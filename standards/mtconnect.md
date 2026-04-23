# MTConnect — Machine Telemetry

**Standard:** ANSI/MTC1.1 through current version 2.2 (2024)
**Maintained by:** AMT – The Association For Manufacturing Technology / MTConnect Institute
**Open source:** Agent, adapter SDK, and schema available at https://github.com/mtconnect
**Status at the shop:** Phase 1 implementation — agents being deployed on priority machines

---

## What the Standard Defines

MTConnect is a royalty-free, open standard for machine tool data communication. It defines an HTTP/XML protocol where a software **agent** (running on a server or edge device) exposes a machine's current and historical state as a structured XML stream. Shop floor software and analytics systems subscribe to this stream via standard HTTP GET requests.

### Architecture

```
MACHINE CONTROLLER
  | (proprietary protocol: Mazak's MAZATROL, Fanuc FOCAS, etc.)
  v
MTConnect ADAPTER
  | (lightweight software translating machine protocol → MTConnect SHDR format)
  v
MTConnect AGENT
  | (standard HTTP/XML server; stores data stream; serves queries)
  v
CONSUMERS: MES, analytics, AI layer, dashboards
```

### Data Items

MTConnect organizes machine data into categories:

**Samples** (continuous, time-series values):
- `AngularVelocity` — spindle speed (RPM)
- `PathFeedrate` — programmed vs. actual feed rate
- `AxisFeedrate` — per-axis actual feed rate
- `Load` — spindle and axis load (% of rated)
- `Temperature` — spindle, coolant
- `AmperageAC/DC` — power consumption
- `Displacement` — linear and angular axis positions

**Events** (state changes, discrete values):
- `Execution` — ACTIVE, FEED_HOLD, INTERRUPTED, PROGRAM_STOPPED, READY, STOPPED
- `ControllerMode` — AUTOMATIC, SEMI_AUTOMATIC, MANUAL, MANUAL_DATA_INPUT, EDIT
- `Program` — active NC program name
- `Block` — currently executing NC program block
- `EmergencyStop` — armed/triggered state
- `WorkOffset` — active work coordinate system
- `ToolNumber` — active tool station
- `SerialNumber` — workpiece serial number (when populated by operator or MES)

**Conditions** (alarm and fault states):
- Normal, Warning, Fault, Unavailable conditions on each component
- Fault codes and text descriptions from the machine controller

### Assets

MTConnect 1.3+ introduced Asset documents — structured XML records that travel with the machine context:

- **CuttingTool:** tool geometry, grade, corner radius, expected life, usage count, coating
- **WorkOrder:** part number, serial number, order number, customer reference, operation reference
- **RawMaterial:** stock dimensions and material type (limited — carries geometry, not lot/cert data)

---

## What the shop Implements

### Machine Fleet

the manufacturer's CNC machines available for MTConnect instrumentation:

| Machine | Controller | MTConnect Path |
|---------|-----------|---------------|
| Mazak Integrex (multi-axis turning/milling) | Mazatrol Matrix | Mazak-native MTConnect agent (built in to controller) |
| Mazak UMC (5-axis milling) | Mazatrol Smooth | Mazak-native MTConnect agent |
| Mazak DVF (5-axis milling) | Mazatrol Smooth | Mazak-native MTConnect agent |
| Mazak NL series (turning) | Mazatrol Matrix | Mazak-native MTConnect agent |
| Mazak Puma series (turning) | Mazatrol Matrix | Mazak-native MTConnect agent |
| Okuma (turning/milling) | OSP-P | Third-party adapter required (Okuma does not ship native MTConnect) |

Mazak controllers include an integrated MTConnect agent accessible at the machine's network IP. Configuration steps:
1. Enable MTConnect on the controller network settings
2. Verify agent is serving at `http://[machine-ip]:5000/current` (or similar)
3. Configure the agent to populate `WorkOrder` asset from production job data (via operator input or automated push from the production system MCP)

### UUID Integration

The critical configuration: the MTConnect `<WorkOrder>` asset must carry the production job UUID and operation UUID so that machine data can be linked back to production system records:

```xml
<WorkOrder assetId="[Operation UUID]">
  <WorkOrderId>[Job UUID]</WorkOrderId>
  <PartNumber>[Customer P/N]</PartNumber>
  <Revision>[Revision]</Revision>
  <SerialNumber>[Serial UUID]</SerialNumber>
</WorkOrder>
```

See `extensions/uuid-discipline.md` for UUID assignment rules and `integration/fulcrum-mtconnect.md` for the the production system → MTConnect data push workflow.

### Data Consumers

- **AI layer (Claude MCP):** Queries MTConnect streams via the the production system MCP server. Enables natural language queries like "how much spindle time was logged on job 25-0412?" or "show me all Execution FEED_HOLD events on the Integrex in the last 30 days."
- **Process correlation:** Link spindle load data to QIF dimensional results for the same serial number to detect if tooling condition correlates with out-of-tolerance dimensions.
- **OEE (Overall Equipment Effectiveness):** Availability, performance, and quality metrics derived from MTConnect data streams.

---

## CM-Layer Gaps

### Gap 1: Material and Certification Data Is Invisible

MTConnect's `<RawMaterial>` asset captures stock dimensions — length, diameter, form type — but not:
- Heat number or lot number
- Mill source or melt country (DFARS compliance)
- MTR or CoC reference
- AMS specification

The machine knows what stock it's cutting; it doesn't know where that stock came from or whether it's DFARS-compliant. This gap is addressed by the production system material lot tracking (see `extensions/material-certs.md`), which maintains the cert chain independently of the machine.

### Gap 2: Outside Processing Operations Are Invisible

Operations performed at outside vendors (heat treat, plating, NDT) produce no MTConnect data. A job's digital thread has:
- MTConnect data for every in-house CNC operation
- A gap in the timeline for every outside processing operation
- No machine data for the receiving/returning of the part from the vendor

This gap is structural — the standard is for CNC machines. The `extensions/outside-processing.md` document defines how outside processing operations are tracked in the production system Pro to fill this gap in the timeline.

### Gap 3: Inspection Is Out of Scope

CMM measurements, in-process dimensional checks, surface roughness measurements, and hardness tests produce no MTConnect data. The quality evidence for a part comes from QIF, not MTConnect. MTConnect's `Condition` data (alarms, faults) provides process quality signals but not part quality results.

### Gap 4: Cost and Scheduling Data

MTConnect reports actual spindle time and machine state but does not:
- Calculate cost (requires labor rates, overhead rates from the production system)
- Compare actual vs. estimated time (requires routing data from the production system)
- Update job status or trigger next-operation scheduling

These require the the production system integration layer — see `integration/fulcrum-mtconnect.md`.

---

## Implementation Reference

- **MTConnect specification:** https://www.mtconnect.org/standard
- **Open-source agent (C++):** https://github.com/mtconnect/cppagent
- **NIST GCR.24-057:** "MTConnect: Enabling the Digital Thread for Manufacturing" — policy-level roadmap and gap analysis
- **NIST AMS 300-12:** MTConnect integration into the digital thread reference architecture
- **Mazak MTConnect setup:** Available in Mazak's machine documentation; the Mazatrol controller exposes a native agent on the machine network
