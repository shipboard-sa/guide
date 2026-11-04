<!--
Purpose: Gap analysis template and the D/F register methodology — Step 3 of the AI workflow.
Version: v1 (2026-11-04) — STUB with methodology section drafted.
Scope: PUBLIC — reusable template + methodology explainer.
Architecture: Takes input from 01-Boat-Survey-Template and 02-Architecture-Tiers, produces
  a gap list that feeds into 04-Reference-BOM.
Invocation: n/a (reference doc)
-->

# Step 3 — Gap Analysis Template

Compare what you have (your [boat survey](01-Boat-Survey-Template.md)) to what your
[architecture tier](02-Architecture-Tiers.md) needs. Your AI session fills this in with you.

## The gap table

For each category, list what your tier requires, what you already have, and what's missing.

| Category | Tier requires | You have | Gap | Priority |
|---|---|---|---|---|
| Data hub (Pi + SignalK) | Raspberry Pi 5 + SignalK | ? | ? | ? |
| N2K gateway | YDEN-02 or equivalent | ? | ? | ? |
| Network (Ethernet to Pi) | wired Ethernet | ? | ? | ? |
| Modbus I/O (Tier 2+) | IO module(s) for non-N2K sensors | ? | ? | ? |
| RS-485 bus (Tier 2+) | gateway or USB adapter | ? | ? | ? |
| Relay pool (Tier 3+) | relay module(s) | ? | ? | ? |
| Alarm panel (Tier 3+) | physical panel | ? | ? | ? |
| Sensor taps (Tier 3+) | parallel tap wiring | ? | ? | ? |
| DIN rail + distribution (Tier 3+) | mounting + power distribution | ? | ? | ? |

Priority: **SAFETY** (must have before anything else) > **FOUNDATION** (other things depend
on it) > **FUNCTION** (does something useful) > **NICE** (adds value but not required).

**STUB** — full gap table templates per tier to be written. For now, your AI session can
generate the gap table from your survey and your tier by reading the reference repos.

---

## The D/F register — your build methodology {#the-df-register}

This is the most important transferable concept from the reference build. The system was
NOT designed top-down — it was discovered iteratively. Your build will be the same.

### What is a D/F register?

Two tables that grow as you build:

**Decisions register (D-nnn):** every choice you make, with context.

| ID | Date | Decision | Reasoning | Alternatives considered |
|---|---|---|---|---|
| D-001 | 2026-12-15 | Use Waveshare IO 8CH for bilge monitoring | Modbus RTU, 8 inputs, DIN-mountable, $45 | Waveshare Relay D (no DIs), Shelly Pro (Wi-Fi only) |
| D-002 | 2026-12-16 | Engine kill via NC relay in series with E-stop | Fail-safe: dead relay = engine runs | NO relay (lose automation), NC shunt (more wiring) |

**Findings register (F-nnn):** every fact you establish, with evidence.

| ID | Date | Finding | Evidence | Impact |
|---|---|---|---|---|
| F-001 | 2026-12-15 | Bilge float switch is a simple dry contact, pulled to 24V | Measured with multimeter: 24V open, 0V when float rises | Can tap directly into Modbus DI (COM = +24V, IN = switch) |
| F-002 | 2026-12-16 | The YDEN-02 draws 50 mA from the N2K bus | Datasheet page 3 | Well within the bus power budget (LEN = 1) |

### Why this matters

- **You WILL discover things you didn't expect.** A wire that isn't where the diagram says.
  A sensor that outputs the wrong signal. A product that doesn't work as advertised. The
  D/F register captures these discoveries so they're not lost.
- **Decisions have context.** Six months from now you'll wonder "why did I use that relay?"
  The D register tells you.
- **Your AI session uses the registers.** When you come back with a new question, the AI
  reads your D/F register and has full context without you re-explaining everything.

### How the reference build uses D/F registers

The reference build has D/F registers in every repo:
- `automation-engine/spec/02-Decisions.md` — 195+ decisions (D-001 through D-195)
- `automation-engine/spec/03-Findings.md` — 340+ findings (F-001 through F-340)
- `wiring-io/wiki/Decisions.md` — hardware decisions (HW-D-)
- `vessel-ops/wiki/Decisions-Register.md` — software decisions (MX-D-)
- `routing/wiki/Decisions-Register.md` — routing decisions (RT-D-)

Plus Open Questions registers (OQ-nnn, ANQ-nn) for things not yet decided. The pattern is:
**every unknown gets an ID and stays visible until resolved.**

### Setting up your registers

Create two files in your project:
- `decisions.md` — start at D-001
- `findings.md` — start at F-001

Your AI session will prompt you to record decisions and findings as you go. The discipline
is: if you learned something or decided something, write it down before moving on.
