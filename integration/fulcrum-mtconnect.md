# the production system ↔ MTConnect Integration

## Integration Goal

MTConnect streams machine state data continuously. the production system manages jobs, operations, and schedules. Neither system alone answers the question: *How much spindle time did job 25-0412, operation 20 actually consume, and how does that compare to the estimate?*

This integration creates the bridge: the production system pushes job context into MTConnect (so machine data is tagged to a job), and MTConnect telemetry flows back into the production system (so actual times and events are captured against the job record).

---

## Data Flow

```
FULCRUM PRO                           MTConnect AGENT
    |                                      |
    | Job 25-0412, Op 20 starts            |
    | Scheduled machine: Mazak UMC         |
    | production job UUID: f91c0d6b-...       |
    | Operation UUID: e4a2f9e1-...         |
    |                                      |
    |--- WorkOrder asset push ------------>|
    |    assetId = Operation UUID          |
    |    WorkOrderId = Job UUID            |
    |    SerialNumber = Serial UUID        |
    |                                      |
    |<-- Execution: ACTIVE ----------------|
    |<-- AngularVelocity: 8500 RPM --------|
    |<-- PathFeedrate: 120 ipm ------------|
    |<-- SpindleLoad: 42% ----------------|
    |                                      |
    | [Operator signals operation done]    |
    |                                      |
    |--- WorkOrder asset update ---------->|
    |    (clear or mark as complete)       |
    |                                      |
    |<-- Execution: PROGRAM_STOPPED -------|
    |                                      |
    | Actual spindle time ingested         |
    | Actual cycle time captured           |
    | Feed-hold events logged              |
```

---

## UUID Push: the production system → MTConnect

The critical configuration step. MTConnect data has no value for job cost or process correlation unless it is tagged to the correct job and operation. The tag is injected via the MTConnect `<WorkOrder>` asset.

### WorkOrder Asset Structure

```xml
<MTConnectAssets>
  <Header creationTime="2025-04-15T08:32:00Z" sender="FulcrumPro" />
  <Assets>
    <WorkOrder assetId="e4a2f9e1-c0d6-b5a3-f8c9-1a3f7c2d18b4"
               timestamp="2025-04-15T08:32:00Z"
               deviceUuid="mazak-umc-001">
      <WorkOrderId>f91c0d6b-5a3f-8c91-a3f7-c2d18b4e4a2f</WorkOrderId>
      <PartNumber>12345-1</PartNumber>
      <Revision>C</Revision>
      <SerialNumber>a3f7c2d1-8b4e-4a2f-9e1c-0d6b5a3f8c91</SerialNumber>
      <RawMaterial>
        <Material>7075-T7351 per AMS 4078</Material>
        <LotId>b5a3f8c9-1a3f-7c2d-18b4-e4a2f9e1c0d6</LotId>
      </RawMaterial>
    </WorkOrder>
  </Assets>
</MTConnectAssets>
```

**Field Mapping:**

| XML Field | UUID Type | Source in the production system |
|-----------|-----------|-------------------|
| `assetId` | Operation UUID | `job.operation[n].id` — the specific routing step |
| `WorkOrderId` | Job UUID | `job.id` |
| `SerialNumber` | Serial UUID | `serial_number.id` (for serialized parts) |
| `LotId` | Material Lot UUID | `material_lot.id` (from material issue record) |
| `PartNumber` | — | `job.part_number` |
| `Revision` | — | `job.revision` |

### Push Mechanism

Three options, in order of preference:

**Option A: Operator entry at machine (Phase 1)**
Operator scans or types the Job Number at the machine control. Mazak Smooth/Matrix controllers support operator-defined variables. The MTConnect adapter maps a designated variable to the WorkOrder asset. Implementation:
- Define a customer variable slot in the Mazak controller (e.g., `MACRO VARIABLE #900 = Job UUID`)
- Configure the MTConnect adapter to read this variable and populate `WorkOrderId`
- Operator scans barcode on traveler at start of operation

**Option B: the production system API push (Phase 1+)**
the production system, via API, pushes the WorkOrder asset directly to the MTConnect agent's asset store when an operator clocks in to an operation in the production system:
- the production system webhook: `operation.start` event → call MTConnect agent PUT /asset
- Requires network access from the production system cloud to MTConnect agent on shop LAN (VPN or local API relay)

**Option C: MCP-mediated push (AI layer)**
Claude, via the production system MCP, reads the job operation start and pushes the WorkOrder asset via a shop-floor API:
- Triggered by the production system operation status change
- Most flexible but adds Claude to the operational path

**Phase 1 recommendation:** Option A for immediate implementation; instrument with Option B when the the production system API relay is available.

---

## MTConnect Ingestion: Machine → the production system

Once machine data is tagged with job/operation UUIDs, telemetry can be correlated and pulled into the production system. The ingestion pipeline:

### What Gets Ingested

| MTConnect Data Item | Ingested Into the production system As |
|--------------------|--------------------------|
| `Execution` state timeline | Operation actual start/end time; feed-hold events |
| `AngularVelocity` (spindle RPM) | Time-series summary (avg, min, max per operation) |
| `PathFeedrate` | Average feedrate vs. programmed feedrate |
| `Load` (spindle %) | Spindle load profile (useful for tool wear correlation) |
| `Program` (NC program name) | Confirms correct program was run; links to revision |
| `ToolNumber` | Tool usage sequence per operation |
| `EmergencyStop` | Fault event log |
| `Condition` faults | Alarm log |

### Calculated Metrics (Fulcrum-Side)

From raw MTConnect data, the production system derives:
- **Actual spindle time** = sum of intervals where `Execution = ACTIVE`
- **Feed hold time** = sum of intervals where `Execution = FEED_HOLD`
- **Setup time** = interval from WorkOrder asset set to first `Execution = ACTIVE`
- **Cycle time** = total interval from first `ACTIVE` to `PROGRAM_STOPPED`
- **OEE Availability** = (scheduled time − downtime) / scheduled time

### Variance Flagging

When actual spindle time deviates from the routing estimate by >20%, flag for review:
- Helps identify NC programs running at reduced feedrate
- Identifies operations where setup took longer than estimated
- Feeds back into routing estimate improvement over time

---

## Machine Fleet Configuration

### Mazak Machines (Native MTConnect)

Mazak Smooth and Matrix controllers include a native MTConnect agent accessible on the machine network. Configuration steps:

1. Enable MTConnect in controller network settings (Mazak service menu)
2. Verify agent is serving at `http://[machine-ip]:5000/current`
3. Test with: `curl http://[machine-ip]:5000/probe` — returns device description XML
4. Configure WorkOrder asset push (see above)
5. Verify agent version supports Assets (MTConnect 1.3+)

**Mazak machine IDs in the production system:**

| Machine | Network IP | MTConnect Device UUID | the production system Equipment ID |
|---------|-----------|----------------------|---------------------|
| Mazak Integrex | [shop network] | mazak-integrex-001 | [Fulcrum UUID] |
| Mazak UMC | [shop network] | mazak-umc-001 | [Fulcrum UUID] |
| Mazak DVF | [shop network] | mazak-dvf-001 | [Fulcrum UUID] |
| Mazak NL (turning) | [shop network] | mazak-nl-001 | [Fulcrum UUID] |
| Mazak Puma (turning) | [shop network] | mazak-puma-001 | [Fulcrum UUID] |

### Okuma (Third-Party Adapter Required)

Okuma OSP-P controllers do not ship with a native MTConnect agent. Options:
- **Okuma App Suite:** Okuma's proprietary software includes an MTConnect adapter as an optional add-on
- **Third-party adapters:** MTConnect-certified adapters from AMT members (e.g., Shop Monitor, OEMsecrets); requires adapter software installed on a PC connected to the Okuma LAN port
- **OpenCASCADE adapter SDK:** Open-source adapter development using MTConnect C++ SDK (`https://github.com/mtconnect/cppagent`)

Phase 1 recommendation: Prioritize Mazak machines; defer Okuma instrumentation to Phase 1 expansion once Mazak integration is validated.

---

## AI Layer Queries (Claude MCP)

With Fulcrum-MTConnect integration active, Claude can answer operational queries against live and historical machine data:

- "How much spindle time did job 25-0412 log on the Mazak UMC yesterday?"
- "Show me all feed-hold events on the Integrex in the last 30 days and which jobs were affected"
- "Which operations are running at less than 80% of programmed feedrate — possible tooling issue?"
- "Compare actual vs. estimated cycle time for part 12345-1 across all jobs in the last 6 months"
- "What is the OEE for the Mazak UMC this week?"
- "Which jobs currently have active WorkOrder assets — who is on what machine right now?"

---

## Phase 1 Verification Checklist

Before declaring Phase 1 complete:

- [ ] MTConnect agent responding on all Mazak machines (`/probe` returns valid XML)
- [ ] WorkOrder asset push working from at least one machine (operator entry or API)
- [ ] Job UUID appearing correctly in MTConnect asset store
- [ ] `Execution` state timeline being captured per job operation
- [ ] Actual spindle time being ingested into the production system and displayed on job record
- [ ] Claude MCP able to query machine data by job number
- [ ] OEE dashboard showing availability/performance data for at least one machine
