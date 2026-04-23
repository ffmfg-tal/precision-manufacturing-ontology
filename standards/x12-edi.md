# X12 EDI — Commercial Transactions

**Standard:** ASC X12 transaction sets; aerospace-specific implementation guide: X12-004010-AIA (AIA/NAS)
**Maintained by:** Accredited Standards Committee X12 (ASC X12)
**Aerospace implementation guidance:** Aerospace Industries Association (AIA) TR-004010-AIA and successor documents
**Status at the shop:** Phase 4 implementation — most current customer/supplier transactions are PDF-based or semi-structured; EDI adoption by the manufacturer's customer base is uneven

---

## What the Standard Defines

X12 EDI (Electronic Data Interchange) is the US/North American standard for machine-readable business document exchange. In aerospace manufacturing, X12 governs the commercial envelope of the supply chain: how customers send orders, how suppliers acknowledge and ship, and how invoices flow.

X12 documents are flat text files with a hierarchical structure of segments and data elements. Modern implementations increasingly use ANSI X12 wrapped in AS2 (Applicability Statement 2) secure HTTP transport, or via VANs (Value-Added Networks) like SPS Commerce or TrueCommerce.

### Relevant Transaction Sets for Aerospace CM

**X12 850 — Purchase Order**
Customer sends to the manufacturer. Contains:
- PO number, PO date, contract reference
- Ship-to and bill-to addresses
- Line items: customer P/N, revision, quantity, unit price, unit of measure, required ship date
- `REF` segments: customer contract number, CAGE code, program/contract reference
- `MSG` free-text notes (where quality clauses are sometimes embedded as unstructured text)
- Payment terms, FOB point, freight terms

**X12 855 — Purchase Order Acknowledgment**
The manufacturer sends to customer. Contains:
- Acceptance or rejection of PO terms
- Confirmed ship date, confirmed quantities
- Any line-level changes or exceptions

**X12 856 — Advance Ship Notice (ASN)**
The manufacturer sends to customer on shipment. Contains:
- Shipment date, carrier, tracking number, BOL number
- Hierarchical shipment structure: shipment → order → line → item
- Per-item: customer P/N, revision, quantity shipped, unit of measure
- `REF` segments: customer PO number, customer part reference, serial/lot numbers
- `SN1` segment: serial and lot number detail at line item level

**X12 810 — Invoice**
The manufacturer sends to customer for payment. Contains:
- Invoice number, invoice date, PO reference
- Line items: quantities, prices, totals
- Payment terms reference
- `REF` segments linking to PO and ASN

**X12 830 — Planning Schedule with Release Capability**
Customer sends to the manufacturer. Contains:
- Forecast quantities by week/period (firmness: firm, planning, or forecast)
- Enables the manufacturer to plan material procurement and capacity
- Not legally binding at forecast horizon; firm releases are the 850

---

## Aerospace-Specific Implementation

The AIA (Aerospace Industries Association) publishes implementation guides for X12 that define mandatory/optional field usage for defense and commercial aerospace. Key aerospace conventions:

**CAGE Code:** Carried in `REF*ZZ` or `REF*FC` segments; identifies the manufacturing source and is required for ITAR/DFARS compliance tracking.

**Contract References:** DFARS contracts carry the contract number in `REF*CT` segments; required for DoD traceability.

**Serial/Lot Numbers on the 856:** The `SN1` segment and associated `REF` segments carry serial numbers for serialized items. For lot-controlled material, lot numbers appear in `REF*LT` segments.

**Revision Levels:** Customer part revision is typically carried in the `PO1` product/item segment on the 850 and in the `SHP` segment on the 856.

---

## What the shop Does with X12 EDI

### Inbound (Customer → Manufacturer)
- **850 Purchase Order:** Primary trigger for job creation in the production system Pro. Customer P/N, revision, quantity, and ship date from the 850 drive production job setup.
- **830 Forecast:** Informs material procurement planning and capacity scheduling.

### Outbound (Manufacturer → Customer)
- **855 Acknowledgment:** Confirms the manufacturer can meet PO requirements (date, price, quantity).
- **856 ASN:** Transmitted at shipment; carries serial/lot numbers that the customer uses to reconcile receipt and update their traceability records.
- **810 Invoice:** Triggers payment; must reconcile to the 850 and 856.

### the production system UUID in X12

The production job UUID is carried in X12 `REF` segments as a cross-reference:
- On the 856 ASN: `REF*ZZ*[Job UUID]` — enables the customer to correlate the shipment to the manufacturer's job record (useful if a quality issue arises post-shipment)
- On the 810 Invoice: same `REF*ZZ` — links invoice to job for the manufacturer's internal cost reconciliation

See `extensions/uuid-discipline.md` for UUID convention.

---

## CM-Layer Gaps

### Gap 1: Quality Clause Flowdowns Have No Standard Representation

The most significant gap. An aerospace PO from a prime contractor carries quality obligations:
- Which AS9100 clauses apply (QA-001 through QA-017 or customer-equivalent)
- Whether FAI is required (and if so, which form type)
- Whether source inspection by the customer's QA is required
- Which material certifications must accompany the shipment (MTR, CoC, DFARS declaration)
- Which special process certs must be included
- Whether a Nadcap-certified vendor is required for specific operations

**None of this is defined in X12 850.** Current practice:
- Quality requirements embedded as `MSG` free-text notes (not machine-readable)
- Separate quality agreement PDF attached to the PO (not in EDI at all)
- Customer-specific quality clauses transmitted via customer portal (outside EDI)

The result: the manufacturer's system cannot programmatically determine from the 850 alone what quality documentation is required for this shipment. A human reads the quality notes and attached PDFs and manually sets up the job requirements.

The `extensions/quality-flowdowns.md` document defines a machine-readable quality clause structure that the shop populates from customer-provided data and uses to drive job requirements in the production system.

### Gap 2: Outside Processing POs Are Sub-Tier — No Standard Linkage

When the manufacturer creates a purchase order to a heat treat vendor for an outside processing operation, that PO is a separate transaction entirely disconnected from the customer's X12 850. There is no standard way to:
- Link the manufacturer's outbound PO to the originating customer PO
- Carry the customer's quality clauses forward to the sub-tier vendor's transaction
- Track the outside processing event in the context of the original job

In practice: the manufacturer attaches quality requirements to sub-tier POs manually. The `extensions/outside-processing.md` document defines the manufacturer data model that maintains this linkage.

### Gap 3: Certification Package Requirements Are Not in the ASN

The X12 856 ASN tells the customer what shipped (quantities, serial numbers, carrier). It does not tell the customer what quality documents are included in the shipment. The cert package — MTR, CoC, FAI report, process certs, inspection records — is sent as a separate PDF packet, a customer portal upload, or physical paperwork in the box.

There is no X12 transaction set for "cert package manifest" that lists which certs are included and links them to the serial numbers they cover. This means the customer cannot programmatically verify cert package completeness at receipt.

### Gap 4: ITAR and Export Control — Not in X12

ITAR-controlled shipments require export licenses, end-user statements, and specific documentation. DFARS requirements for specialty metals must be declared. Neither ITAR classification of the article nor DFARS metal origin is carried in X12 as a structured field. Compliance is documented separately.

---

## Integration Points

| X12 Data | Maps To | the production system Entity |
|----------|---------|----------------|
| 850 PO Number | Job / Sales Order | production job `customer_po_number` |
| 850 Line Item (P/N, Rev, Qty, Date) | Job creation | production job header fields |
| 850 MSG notes (quality clauses) | Quality plan | `extensions/quality-flowdowns.md` structure |
| 856 Serial/Lot numbers | Serialization | the production system serial number records |
| 856 REF*ZZ (Job UUID) | Cross-reference | production job UUID |
| 810 Invoice amounts | Billing | the production system invoice record |

---

## Resources

- **ASC X12:** https://x12.org — standard publications (subscription required for full standard; summaries available)
- **AIA EDI implementation guide:** X12-004010-AIA-E4 — aerospace-specific implementation instructions
- **ANSI X12 transaction set 850:** https://x12.org/codes/850
- **EDI reference at-a-glance:** Many publicly available tutorials describe X12 segment structure; EDI Notepad and EDICAT are useful free parsers for inspecting X12 files
