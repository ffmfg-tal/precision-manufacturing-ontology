# ffmfg-core — Business Spine Database Design

**Status:** v1.0 — approved design, ready for implementation planning
**Date:** 2026-04-24
**Audience:** Tal, Lex (review); Lino (quality app), Braxton (supply chain) as downstream consumers; future agent-team engineers
**Scope:** Canonical business-object data store and API — the replacement for Fulcrum's role as the "business-object spine"

---

## 1. Purpose

`ffmfg-core` is the canonical store for FFM's core business objects — items, customers, vendors, purchase orders, sales orders, jobs, routings — and the API through which every other FFM system reads and mutates them. It is the programmatic implementation of the contract-manufacturer-ontology.

### What it replaces
Fulcrum Pro's role as the authoritative record for business objects. Fulcrum remains the operating ERP during migration; `ffmfg-core` stands up in parallel, imports from Fulcrum on a schedule, and becomes the system of record once cutover completes.

### What it serves
- **Email agents** (`orders@`, `purchasing@`, `quality@`, etc. per `agents-at.md`) — correlation verification and canonical reads/writes via MCP
- **shop-floor-mes** — job/routing/operation data for scheduling (replaces today's Fulcrum sync)
- **Lino's quality app** — quality entities (FAI, NCR, characteristic, measurement_result, PPAP)
- **Braxton's supply chain app** — supplier, ASL, PO, receipt entities
- **Future agents and internal tooling** — via the same MCP/REST contract

### What it is not
- Not a scheduling engine (that's the MES)
- Not a quality UI (that's Lino's app)
- Not a CAM/PLM system
- Not a replacement for Fulcrum's accounting module (out of scope per `project_replacement_scope.md`)

---

## 2. Architectural principles

1. **Ontology is canon.** Entity names, field names, types, relationships, enums, and invariants are defined once in `contract-manufacturer-ontology/schemas/*.yaml`. Code generation produces TypeScript types and Zod validators; migrations are hand-written and numbered but must validate against the ontology.
2. **Every mutation is an event.** No direct writes to entity tables — all state transitions go through `commit(event, principal, correlation_id)`. Events are append-only, chain-hashed, and externally anchored.
3. **Time-travel is a feature, not a bolt-on.** Every entity has a history table. "What did PO-5678 look like on 2026-03-15?" is a normal query.
4. **Infrastructure is abstracted.** Domain code never imports Cloudflare bindings directly. `Database`, `BlobStore`, `SecretStore`, `Clock` are interfaces; platform adapters live in `src/infra/`. Port to AWS GovCloud is a matter of swapping adapters.
5. **CMMC L2 / NIST SP 800-171 capable from day 1.** The system is built to handle CUI once migrated to a GovCloud-equivalent environment. No CUI enters the DB until that migration completes, but the controls are wired now.
6. **Default-deny, explicit-allow.** No unauthenticated callers. Every principal (user, agent, sync job) has a scoped policy. Every call is audited.
7. **Idempotent by design.** Every mutation is keyed by `(principal, correlation_id, event_type)`. Retries are safe; duplicates are rejected.
8. **Raw SQL, typed thinly.** No ORM. A small query helper gives parameter binding and row-to-type mapping. Migrations and queries are legible to anyone who knows SQL. Claude is the primary author and maintainer of the SQL surface.
9. **One thing, one place.** MCP tools, REST routes, and the domain service layer all call the same handlers — no shadow implementations, no duplicate business logic.

---

## 3. System architecture

```
 ┌────────────────────────────────────────────────────────────────────┐
 │ ffmfg-core (Cloudflare Worker on CF; portable to AWS Lambda)        │
 │ Served at core.ffmfg.com                                            │
 │                                                                     │
 │  ┌────────────────┐   ┌───────────────┐   ┌──────────────────────┐  │
 │  │ REST API       │   │ MCP server    │   │ Sync / ingest        │  │
 │  │ /api/v1/*      │   │ (tools)       │   │ (Fulcrum → events)    │  │
 │  └───────┬────────┘   └───────┬───────┘   └──────────┬───────────┘  │
 │          │                    │                      │              │
 │          └────────────────────┼──────────────────────┘              │
 │                               ▼                                     │
 │  ┌───────────────────────────────────────────────────────────────┐  │
 │  │ Domain service layer                                          │  │
 │  │ • One module per entity (items, purchase_orders, jobs, ...)   │  │
 │  │ • All writes route through commit(event, principal, corrId)   │  │
 │  │ • Authorization enforced here, not at the edge                │  │
 │  └───────────────┬───────────────────────────┬───────────────────┘  │
 │                  ▼                           ▼                      │
 │  ┌──────────────────────────┐   ┌───────────────────────────────┐   │
 │  │ Database (interface)     │   │ BlobStore (interface)         │   │
 │  │ D1 adapter (today)       │   │ R2 adapter (today)            │   │
 │  │ Postgres adapter (future)│   │ S3 adapter (future)           │   │
 │  └──────────────────────────┘   └───────────────────────────────┘   │
 └──────────────────────────────────────────────────────────────────────┘
                      │                              │
                      ▼                              ▼
          ┌────────────────────────┐    ┌──────────────────────────────┐
          │ D1                     │    │ R2                           │
          │ • events               │    │ • attachments by sha256      │
          │ • <entity>_history     │    │ • certs, FAI packages,       │
          │   (SCD2 per entity)    │    │   drawings, rendered PDFs    │
          │ • access_policies      │    │ • audit_anchors (WORM-locked)│
          │ • principals           │    │                              │
          └────────────────────────┘    └──────────────────────────────┘
                      ▲                              ▲
                      │ MCP / REST                   │ REST
                      │                              │
         ┌────────────┴─────────┐    ┌───────────────┴──────────────────┐
         │ Email agents         │    │ MES / Lino's app /               │
         │ (orders@, etc.)      │    │ Braxton's app / humans via UI    │
         └──────────────────────┘    └──────────────────────────────────┘
```

### Runtime stack

| Layer | Choice | Rationale |
|---|---|---|
| Runtime | Cloudflare Workers | Matches MES and fulcrum-pro-mcp; portable via infra abstraction |
| HTTP | Hono | Lightweight, CF-native, trivial to adapt to Lambda |
| Database (today) | D1 | Matches existing stack; sufficient for projected scale |
| Database (future) | Postgres (RDS/Aurora on AWS) | CMMC-compliant target environment |
| Blob (today) | R2 | S3-compatible API; portable |
| Blob (future) | S3 with Object Lock in GovCloud | CMMC-compliant target |
| Validation | Zod | Generated from ontology YAML; runtime + static type coverage |
| MCP SDK | `@modelcontextprotocol/sdk` | Standard pattern from fulcrum-pro-mcp |
| Tests | Vitest + `@cloudflare/vitest-pool-workers` | Runs tests in Worker runtime; matches existing projects |
| SQL | Raw SQL + thin `query()` helper | No ORM; SQL is legible and debuggable |

---

## 4. Immutability, audit, and integrity

Three artifacts, one design intent: every state change is provable.

### 4.1 The `events` table

Append-only, never updated, never deleted. Every mutation in the system writes exactly one row here before any entity table is touched.

```sql
CREATE TABLE events (
  id                TEXT PRIMARY KEY,           -- ULID (sortable, generated server-side)
  entity_type       TEXT NOT NULL,              -- 'purchase_order' | 'job' | ...
  entity_id         TEXT NOT NULL,              -- UUID of the target entity
  event_type        TEXT NOT NULL,              -- 'created' | 'updated' | 'status_changed' | 'deleted'
  principal_type    TEXT NOT NULL,              -- 'user' | 'agent' | 'system_sync'
  principal_id      TEXT NOT NULL,              -- user UUID, agent name, or sync job name
  occurred_at       TEXT NOT NULL,              -- ISO 8601 UTC (server clock)
  correlation_id    TEXT,                       -- e.g. email message_id, sync run id
  reason            TEXT,                       -- freeform: "po_ack from Relativity via email"
  payload           TEXT NOT NULL,              -- JSON: full new entity state (not a diff)
  prior_event_id    TEXT,                       -- previous event for this entity (nullable for 'created')
  payload_hash      TEXT NOT NULL,              -- sha256(canonical_json(payload))
  chain_hash        TEXT NOT NULL,              -- sha256(prior.chain_hash || payload_hash)
  ingest_version    TEXT NOT NULL,              -- codebase version that wrote this event
  FOREIGN KEY (prior_event_id) REFERENCES events(id)
);

CREATE UNIQUE INDEX idx_events_idempotency
  ON events(principal_type, principal_id, correlation_id, event_type, entity_type, entity_id)
  WHERE correlation_id IS NOT NULL;

CREATE INDEX idx_events_entity ON events(entity_type, entity_id, occurred_at);
CREATE INDEX idx_events_correlation ON events(correlation_id);
CREATE INDEX idx_events_principal ON events(principal_type, principal_id, occurred_at);
```

**Rules:**
- `payload` is always the full new state, never a diff. Replay requires no prior context.
- `prior_event_id` is the most recent event for the same `(entity_type, entity_id)`. Looked up inside the write transaction under the entity's row lock.
- Events are never updated or deleted. Corrections are new `corrected` events that reference the wrong event's id.

**Canonical JSON (normative):**
- UTF-8 byte stream output.
- Object keys sorted lexicographically by UTF-16 code unit.
- No whitespace between tokens.
- Strings encoded per RFC 8259 with no escape-set choice ambiguity: `"`, `\`, control characters < 0x20, and `\u2028`/`\u2029` are the only escaped codepoints; all other characters emitted literally.
- Numbers: integers as decimal with no leading `+` or leading zeros; non-integer numbers are disallowed in event payloads (use string representations for fixed-point quantities — dollars as `"12345"` cents, dimensions as strings with units).
- A `canonicalJson()` helper implements this. Property-tested against a fixture set of equivalence-class pairs.

**Chain hash (normative):**
- `payload_hash = SHA-256( canonical_json_bytes( payload ) )`, output as lowercase hex (64 chars).
- `chain_hash = SHA-256( hex_decode(prior_chain_hash) || hex_decode(payload_hash) )`, output as lowercase hex. `||` is byte concatenation of the 32-byte hash digests.
- For the first event of an entity, `prior_chain_hash` is the 32-byte all-zero string (hex: 64 zeros).
- A conformance test vector file (`tests/fixtures/chain-vectors.json`) pins example inputs and expected outputs. Any adapter or reimplementation must pass it.

### 4.2 Per-entity history tables (SCD2)

Every entity has its own table structured as a SCD2 history. Current state is `valid_to IS NULL`.

Example:
```sql
CREATE TABLE purchase_orders_history (
  id                TEXT NOT NULL,              -- entity UUID (not unique in this table)
  valid_from        TEXT NOT NULL,              -- ISO 8601 UTC
  valid_to          TEXT,                       -- NULL = current; set on supersession
  event_id          TEXT NOT NULL,              -- the event that produced this version
  -- ontology fields for purchase_order:
  po_number         TEXT NOT NULL,
  supplier_id       TEXT NOT NULL,
  status            TEXT NOT NULL,
  po_kind           TEXT NOT NULL,              -- 'standard' | 'sub_tier'
  ...
  PRIMARY KEY (id, valid_from),
  FOREIGN KEY (event_id) REFERENCES events(id)
);

CREATE UNIQUE INDEX idx_po_current ON purchase_orders_history(id) WHERE valid_to IS NULL;
CREATE INDEX idx_po_supplier ON purchase_orders_history(supplier_id) WHERE valid_to IS NULL;
CREATE INDEX idx_po_status ON purchase_orders_history(status) WHERE valid_to IS NULL;
CREATE INDEX idx_po_number ON purchase_orders_history(po_number);  -- historical lookups
```

**Rules:**
- Reads for "current" filter on `valid_to IS NULL` via a partial unique index (D1/SQLite support this).
- Reads for "state at time T" use `valid_from <= T AND (valid_to IS NULL OR valid_to > T)`.
- All writes happen inside a transaction: insert event → update prior row's `valid_to` → insert new row. Orchestrated by the `commit()` helper.
- There are no `<entity>_current` tables. History is both the audit and the serving layer.

### 4.3 External audit anchor (tamper-evidence)

The chain_hash is tamper-evident only if verified externally. A scheduled worker (hourly) takes the latest `chain_hash` per entity_type and writes a signed anchor record to R2 under Object Lock (WORM).

```
r2://ffmfg-core-audit-anchors/
  anchors/
    {YYYY-MM-DD}/
      {HH}-{entity_type}.json      # { anchor_time, last_event_id, chain_hash, signature }
      {HH}-manifest.json           # { entity_type → anchor_file_key, ... }
```

Object Lock prevents deletion or modification for the retention period (minimum 7 years per CMMC AU-11 guidance). If an attacker tampers with the events table, the chain breaks against the last externally anchored hash.

**Signing key management (SC-12 / SC-13):**
- Algorithm: Ed25519. 32-byte private key, 32-byte public key.
- **Today (pre-GovCloud):** private key stored in Cloudflare Secrets, bound to the anchor worker only; rotated annually (new key at year boundary, old public key retained in the manifest for verification of older anchors).
- **Post-GovCloud:** private key in AWS KMS as a customer-managed CMK with `Sign` permission granted only to the anchor Lambda's IAM role. KMS audit log captures every sign call.
- **Public key publication:** `r2://ffmfg-core-audit-anchors/manifest.json` contains `{ key_id, public_key_hex, active_from, active_to }` for every key the anchor worker has ever used. Anchors cite the `key_id` that signed them. Anyone with read access to the bucket can independently verify the chain.
- **Compromise recovery:** rotate key; write a `key_compromise` event into the events table; re-anchor the current chain tip under the new key; the manifest retains the compromised key's `active_from`/`active_to` so prior anchors remain verifiable for forensic purposes.

### 4.4 What goes in R2 (not D1)

Content-addressed binary artifacts:
- FAI packages, MTRs, process certs, CoCs
- Drawings (PDF, STEP, DXF, SLDPRT)
- Outbound PDFs (rendered POs, quotes, packing slips)
- Audit anchors (§4.3)
- Email `.eml` files when they correlate to a canonical entity (the agents-at.md substrate handles broader email archive; anchored copies land here for the events that reference them)

D1 carries the sha256 and metadata (filename, content_type, size, provenance); R2 holds the bytes. Same sha256 = single stored copy.

---

## 5. Database conventions

### 5.1 Identity
- All entity IDs are UUIDv7 (time-ordered, aligns with the ontology's UUID discipline).
- Event IDs are ULIDs (sortable, 26-char base32).
- External IDs (Fulcrum IDs, PO numbers, SO numbers, part numbers) are stored as separate columns. They are human-readable keys, not primary keys.

### 5.2 Naming
- Tables: `<entity>_history` (all entity tables are SCD2; no unsuffixed mutable tables).
- System tables: `events`, `principals`, `access_policies`, `sync_runs`.
- Views (optional): `<entity>_current` as `SELECT * FROM <entity>_history WHERE valid_to IS NULL` — provided for ergonomics but callers should prefer indexed table access.

### 5.3 Foreign keys
Cross-entity references use `<entity>_id` pointing to the UUID. Referential integrity is enforced by application code, not by SQL FK constraints — because the target entity may have newer versions and SQLite FK semantics across partial unique indexes are brittle. Application-level integrity is covered by tests.

### 5.4 Enums
Enum values live in the ontology YAML and are regenerated as TypeScript string literal unions + D1 CHECK constraints.

### 5.5 Time
All timestamps are ISO 8601 UTC strings. No epoch integers, no local time, no timezone ambiguity. A `Clock` interface abstracts `now()` so tests are deterministic.

---

## 6. Schema authoring workflow

```
contract-manufacturer-ontology/schemas/*.yaml   (canon)
                │
                ▼
         codegen tool (ffmfg-core/tools/codegen/)
                │
                ├──> ffmfg-core/src/schemas/*.ts      (Zod + TS types)
                └──> ffmfg-core/src/enums/*.ts        (string literal unions)

ffmfg-core/migrations/NNNN_<slug>.sql               (hand-written, numbered)
```

### Rules
- Codegen is **types-only**. It does not emit migrations.
- Every PR that changes a YAML schema must include either: (a) a new numbered migration that conforms the DB to the new ontology, or (b) a note that the change is non-breaking (e.g., new optional field added).
- CI validates: generated Zod schema against the latest `<entity>_history` table shape. Drift fails the build.
- Migrations are additive-only in production (never `DROP COLUMN`). Deprecations are tracked in the ontology YAML with a `deprecated_in` field and a migration plan.

---

## 7. API surface

### 7.1 REST

- Base: `https://core.ffmfg.com/api/v1/`
- Versioned in the path. v1 never breaks; new shape = `/api/v2/`.
- JSON in, JSON out. Zod validation at every route.
- Standard error envelope: `{ error: { code, message, details } }`.

Routes follow CRUD conventions:
```
GET    /api/v1/purchase_orders                      — list with filters
GET    /api/v1/purchase_orders/:id                  — current state
GET    /api/v1/purchase_orders/:id/history          — full SCD2 trail
GET    /api/v1/purchase_orders/:id?at=<iso8601>     — state at point in time
POST   /api/v1/purchase_orders                      — create
PATCH  /api/v1/purchase_orders/:id                  — update (triggers event)
POST   /api/v1/purchase_orders/:id/status           — explicit state transitions
```

### 7.2 MCP tools

Separate server mode exposing the same domain services as MCP tools. Tool names are versioned:

```
get_purchase_order_v1(po_number | po_id) → PurchaseOrder
list_purchase_orders_v1(filters) → PurchaseOrder[]
upsert_purchase_order_v1(po: PurchaseOrderInput, correlation_id) → PurchaseOrder
verify_correlation_v1(ref_text) → { verified: bool, entity_type, entity_id }
...
```

`verify_correlation_v1` is the critical tool for the email classifier — given a string like "PO-5678" or "Q-2261", it verifies the reference exists and returns its canonical form. This replaces the Fulcrum MCP's correlation role.

### 7.3 Versioning policy
- Tool and route versions in the name (`_v1` suffix, `/api/v1/` prefix).
- A tool's shape is frozen once any external consumer uses it.
- New shape = new version; old version supported for a deprecation window (minimum 6 months).
- Version policy doc lives in `docs/api-versioning.md` (to be written during implementation).

---

## 8. Authorization model

### 8.1 Principals
```sql
CREATE TABLE principals (
  id              TEXT PRIMARY KEY,     -- UUID
  principal_type  TEXT NOT NULL,        -- 'user' | 'agent' | 'system_sync'
  name            TEXT NOT NULL,        -- display name
  entra_subject   TEXT,                 -- Entra ID sub for user principals
  service_key_hash TEXT,                -- bcrypt hash of service key for agents
  status          TEXT NOT NULL,        -- 'active' | 'revoked'
  created_at      TEXT NOT NULL,
  revoked_at      TEXT
);
```

- **Users**: authenticated via Entra ID SSO (OIDC). Session token minted by the Worker; contains the principal_id. User entity shape mirrors `schemas/user.yaml`; the `principals` table holds the auth-layer projection (Entra subject, status, service credentials). A user's canonical data (citizenship, training records, signatory authority) lives in the `user_history` SCD2 table per the ontology.
- **Agents**: service accounts with long-lived service keys stored in CF Secrets (or AWS Secrets Manager post-migration). One per intent agent (`agent/po_ack`, `agent/rfq_response`, etc.).
- **System sync**: bootstrap and periodic sync jobs (`system_sync/fulcrum_import`).

**Overlap with shop-floor-mes `operators` table:** MES has its own `operators` table seeded from Fulcrum timer history, enriched with skill level and default machine. Post-ffmfg-core, MES `operators` becomes a scheduling-layer enrichment projection keyed by `user_id` → the canonical ffmfg-core user. MES keeps its enrichment fields; ffmfg-core owns identity and the canonical user record. Similar pattern applies to Lino's and Braxton's existing user data: they keep local enrichment, ffmfg-core owns identity. A migration script maps existing MES operator rows to ffmfg-core users during bootstrap.

### 8.2 Policies
```sql
CREATE TABLE access_policies (
  id              TEXT PRIMARY KEY,
  principal_id    TEXT NOT NULL,
  entity_type     TEXT NOT NULL,
  action          TEXT NOT NULL,        -- 'read' | 'write' | 'admin'
  scope_filter    TEXT,                 -- JSON: optional field-level or row-level filter
  granted_by      TEXT NOT NULL,        -- principal_id of grantor
  granted_at      TEXT NOT NULL,
  revoked_at      TEXT,
  FOREIGN KEY (principal_id) REFERENCES principals(id)
);
```

Default: deny. A request is authorized only if an active policy exists for (principal, entity_type, action).

Policy evaluation happens in the domain service layer — not at the HTTP edge — so MCP tools, REST routes, and internal callers all enforce the same rules.

### 8.3 ITAR / CUI scoping (prepared, not yet active)
Certain entities (part_specification with `itar_classified = true`, export_classification, certain drawings) will require a "CUI-cleared" policy scope post-GovCloud migration. Schema reserves the scope column; enforcement stays permissive until the migration cuts over.

### 8.4 Audit
Every successful or denied request writes to `events` with `event_type = 'access_granted' | 'access_denied'` and the full request context. Authorization is itself an event stream.

---

## 9. Idempotency & retry semantics

- Every write MUST carry a `correlation_id` (email message_id, sync run id, MCP request id).
- The `idx_events_idempotency` unique index on `(principal_type, principal_id, correlation_id, event_type, entity_type, entity_id)` rejects duplicate writes.
- On duplicate-key error, the domain layer returns the result of the original event (fetched from events table) — same response shape as if it had just succeeded. Safe retry.
- Callers without a correlation_id receive a server-generated ULID and should retry with it on failure.

---

## 10. Infrastructure abstraction layer

### 10.1 Interfaces (in `src/infra/`)

```typescript
// src/infra/database.ts
export interface Database {
  query<T>(sql: string, params: unknown[]): Promise<T[]>;
  queryOne<T>(sql: string, params: unknown[]): Promise<T | null>;
  transaction<T>(fn: (tx: Database) => Promise<T>): Promise<T>;
}

// src/infra/blob.ts
export interface BlobStore {
  put(key: string, body: ReadableStream | ArrayBuffer, opts?: { contentType?: string; objectLock?: boolean }): Promise<{ sha256: string }>;
  get(key: string): Promise<ReadableStream | null>;
  head(key: string): Promise<{ sha256: string; size: number; contentType: string } | null>;
}

// src/infra/secrets.ts
export interface SecretStore {
  get(name: string): Promise<string>;
}

// src/infra/clock.ts
export interface Clock {
  now(): string;  // ISO 8601 UTC
}

// src/infra/id.ts
export interface IdGenerator {
  ulid(): string;
  uuid(): string;  // UUIDv7
}
```

### 10.2 Platform adapters

- `src/infra/d1/` — D1 Database impl, parameter binding, transaction semantics
- `src/infra/r2/` — R2 BlobStore impl including Object Lock config
- `src/infra/cf-secrets/` — CF env bindings as SecretStore
- (Future) `src/infra/postgres/` — pg client for AWS Aurora
- (Future) `src/infra/s3/` — AWS SDK v3 S3 client
- (Future) `src/infra/aws-secrets/` — AWS Secrets Manager

### 10.3 Rule
Grep check in CI: no file outside `src/infra/` may import from `@cloudflare/workers-types`, `aws-sdk`, or reference `env.DB` / `env.BUCKET` by name. Violations fail the build. This is the portability guarantee.

---

## 11. Fulcrum bootstrap and sync

### 11.1 Bootstrap (one-time)
- A `system_sync/fulcrum_bootstrap` job reads every in-scope entity from Fulcrum via the existing MCP/REST client.
- For each Fulcrum record, emit a `created` event into ffmfg-core with `principal_type = 'system_sync'`, `correlation_id = 'fulcrum_bootstrap_<run_id>_<fulcrum_id>'`. Idempotency prevents double-import on retry.
- Fulcrum's ID is stored as an `external_ref` field; ffmfg-core assigns its own UUIDv7 as the canonical id.
- Bootstrap runs in shadow mode first: writes to a parallel database, compared against Fulcrum's source state, before being promoted.

### 11.2 Ongoing sync (during migration)
- A 5-minute cron polls Fulcrum for changes (mtime or equivalent).
- Each changed entity writes a new event (`updated` with the full current state).
- Sync is one-way (Fulcrum → ffmfg-core) until cutover. Writes from consumers (MES, email agents) during this period are allowed to ffmfg-core only; cutover completes when all consumers stop writing to Fulcrum.

**Email classifier correlation order (during the parallel period):**
1. Classifier calls `ffmfg-core.verify_correlation_v1(ref_text)` first.
2. If `verified=true`, done.
3. If `verified=false`, fall through to the `fulcrum-pro-mcp` correlation tool. If Fulcrum verifies the reference and ffmfg-core does not, a `sync_gap` event is written into ffmfg-core (principal `system_sync/classifier_gap_detector`, correlation_id = email message_id) so the gap is visible in the audit stream and can trigger an out-of-band Fulcrum pull for the missing entity.
4. Fallback window closes at cutover. Post-cutover, only ffmfg-core is queried; Fulcrum correlation tool is decommissioned in the email agents' config.

### 11.3 Cutover
- Sync reverses direction: ffmfg-core becomes SoR; a reverse-sync writes back to Fulcrum for the remaining Fulcrum-dependent workflows until those are also migrated.
- A dedicated runbook (separate doc) covers the cutover sequence.

---

## 12. Testing strategy

### 12.1 Unit tests
- Every domain service module has unit tests for: event construction, payload hashing, policy evaluation, invariant checks.
- Pure functions: `applyEvent(priorState, event) → newState` is property-testable. Every entity has property tests asserting apply → replay equivalence.

### 12.2 Integration tests
- Run in `@cloudflare/vitest-pool-workers` against a local D1 instance.
- Every MCP tool and REST route has at least one integration test exercising the full write path → events row → history row → read-back.
- Idempotency test: same correlation_id twice returns the same result.
- Chain integrity test: write 100 events, tamper with one payload, chain verification detects it.
- Authorization test: unauthorized principal receives 403 and writes an `access_denied` event.

### 12.3 Contract tests
- A `contract-tests/` directory holds shape assertions for every MCP tool and REST route. These run in CI against both local dev and the deployed preview environment.
- Contracts serve as the versioning guarantee: if a contract test for `get_purchase_order_v1` breaks, the v1 contract is being violated and the change is rejected.

### 12.4 Ontology conformance
- CI runs codegen; if generated types differ from the committed `src/schemas/*.ts`, the build fails.
- A shape-conformance test compares every `<entity>_history` table's columns against the generated Zod schema. Column drift fails CI.

### 12.5 Replay verification
- A periodic job rebuilds every entity's current state from events alone and compares to `<entity>_history WHERE valid_to IS NULL`. Any mismatch is an incident.

---

## 13. Observability

- **Logs**: structured JSON, one line per request. Fields: principal_id, route, correlation_id, event_ids_written, latency, status.
- **Metrics**: request rate, error rate, latency p50/p95/p99, events-written-per-minute, chain-anchor-lag. Exported via CF Analytics Engine today; AWS CloudWatch post-migration.
- **Audit stream**: the `events` table is the primary audit log. A read-only MCP tool (`query_audit_events_v1`) allows authorized callers to slice it.
- **Alerts**: chain-anchor-lag > 2 hours; replay mismatch; unauthorized access spike; sync lag > 15 minutes.

---

## 14. CMMC L2 control coverage (preparation)

The system is designed to satisfy NIST SP 800-171 controls required for CMMC Level 2. Mapping (abbreviated):

| Control family | How ffmfg-core satisfies it |
|---|---|
| **AC (Access Control)** | §8: principals + policies + default-deny; Entra ID for users; service keys for agents |
| **AU (Audit & Accountability)** | §4: events table + external WORM anchors; AU-2/AU-3/AU-11 all covered |
| **CM (Configuration Management)** | §6: ontology-as-canon; migrations numbered and in git; ingest_version on every event |
| **IA (Identification & Authentication)** | §8: Entra ID SSO (IA-2); service keys rotated via secrets manager (IA-5) |
| **SC (System & Communications Protection)** | TLS throughout; data-at-rest encryption via D1/R2 today, S3/RDS post-migration |
| **SI (System & Information Integrity)** | §4: chain_hash + external anchor; replay verification detects integrity violation |
| **MP (Media Protection)** | Deferred — no CUI until GovCloud migration |
| **PE (Physical Protection)** | Deferred — inherited from hosting provider post-migration |

Pre-migration, the system runs on commercial CF with no CUI. Post-migration, the same architecture deploys to GovCloud with no domain-code changes required (only infra adapters swap).

---

## 15. Phase 1 scope — first implementation

The first shipping cut delivers the minimum spine needed to unblock the email agents.

### Entities (hand-written migrations, aligned to existing ontology schemas)
1. `principal` — authorization subjects
2. `access_policy` — authorization rules
3. `item` — part/assembly identity (references `schemas/part.yaml`)
4. `customer` — (`schemas/customer.yaml`)
5. `supplier` — (`schemas/supplier.yaml`)
6. `customer_purchase_order` — inbound PO from customers (`schemas/customer_purchase_order.yaml`)
7. `sales_order` — (`schemas/sales_order.yaml`)
8. `purchase_order` — outbound PO to suppliers (`schemas/purchase_order.yaml`)
9. `job` — production run (`schemas/job.yaml`, `job` entity)

(Routing, operations, work orders, quality entities come in Phase 2.)

### MCP tools (for the email classifier and `po_ack` / `order_entry` agents)
- `verify_correlation_v1(ref_text)` — the workhorse for the classifier
- `get_item_v1`, `list_items_v1`
- `get_customer_v1`, `get_supplier_v1`
- `get_purchase_order_v1`, `list_purchase_orders_v1`, `upsert_purchase_order_v1`
- `get_customer_purchase_order_v1`, `upsert_customer_purchase_order_v1`
- `get_sales_order_v1`, `upsert_sales_order_v1`
- `get_job_v1`, `list_jobs_v1`
- `query_audit_events_v1`

### REST routes
Mirror of the MCP tools under `/api/v1/`.

### Fulcrum sync
Bootstrap + ongoing 5-minute sync for the nine entities above.

### Infrastructure
- CF Worker deployed at `core.ffmfg.com` (DNS cutover via Braxton)
- D1 database `ffmfg-core` created
- R2 bucket `ffmfg-core-blobs` (attachments) and `ffmfg-core-audit-anchors` (WORM-locked)
- Entra ID app registration for SSO
- Secrets: Fulcrum API token, Entra client secret, service keys for initial agents

### Out of scope for Phase 1
- Routing / operation / work_order entities
- Quality entities (FAI, NCR, characteristic, measurement_result)
- Shipments, receipts
- Outbound PDF generation (that's in the email-agents stack, not here)
- ITAR / CUI enforcement activation (policy scope prepared but permissive)
- AWS adapter implementations

---

## 16. Repository layout

```
ffmfg-core/                                 (new repo, sibling to shop-floor-mes)
├── migrations/
│   └── NNNN_<slug>.sql                     (hand-written, numbered)
├── tools/
│   └── codegen/                            (ontology YAML → TS/Zod)
├── src/
│   ├── index.ts                            (Worker entry point)
│   ├── schemas/                            (generated Zod + TS types)
│   ├── enums/                              (generated string literal unions)
│   ├── infra/
│   │   ├── database.ts                     (interface)
│   │   ├── blob.ts                         (interface)
│   │   ├── secrets.ts                      (interface)
│   │   ├── clock.ts                        (interface)
│   │   ├── id.ts                           (interface)
│   │   ├── d1/                             (CF D1 adapter)
│   │   ├── r2/                             (CF R2 adapter)
│   │   └── cf-secrets/                     (CF env secrets adapter)
│   ├── domain/
│   │   ├── commit.ts                       (the central write path)
│   │   ├── events.ts                       (event construction + hashing)
│   │   ├── canonical-json.ts               (deterministic JSON)
│   │   ├── auth.ts                         (principal resolution + policy eval)
│   │   └── entities/
│   │       ├── item/
│   │       ├── purchase_order/
│   │       ├── sales_order/
│   │       ├── job/
│   │       └── ...                         (one dir per entity)
│   ├── api/
│   │   ├── rest/                           (Hono routes)
│   │   └── mcp/                            (MCP server + tool registrations)
│   ├── sync/
│   │   └── fulcrum/                        (bootstrap + ongoing sync)
│   └── jobs/
│       ├── anchor.ts                       (hourly audit anchor publisher)
│       └── replay-verify.ts                (periodic replay verification)
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── contract/
│   └── conformance/
├── wrangler.jsonc
├── package.json
├── tsconfig.json
└── CLAUDE.md
```

---

## 17. Open questions deferred to the implementation plan

1. **Entity scope ordering within Phase 1.** Nine entities is still a lot for the first cut; the implementation plan should sequence them so Fulcrum bootstrap can run partially (item + customer + supplier first, then PO/SO/job).
2. **Backpressure and CF Worker limits.** 50-subrequest cap affects bulk sync. Needs explicit chunking strategy in the sync module.
3. **Schema alignment with Lino and Braxton.** Before finalizing `purchase_order` and quality entity migrations, confirm field shapes with both teams. They are downstream consumers who will notice breaking changes first.
4. **Replay-rebuild story in practice.** Does `replay-verify` need to run on every deploy, nightly, or weekly? Cost and D1 read-rate affect the answer.
5. **Secondary index strategy for history tables.** Partial indexes on `valid_to IS NULL` cover current-state reads. Historical queries may need separate indexes; determine per-entity during implementation.
6. **Cutover criteria.** Concrete measurable gates for "Fulcrum is no longer SoR" — probably: zero writes to Fulcrum from internal systems for N days, all consumers on ffmfg-core, reverse-sync stable.

---

*End of design. Implementation plan follows.*
