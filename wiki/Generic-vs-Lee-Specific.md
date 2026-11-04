<!--
Purpose: Explicit guide to what's transferable vs. what's Lee's configuration in the reference repos.
Version: v1 (2026-11-04)
Scope: PUBLIC — cross-cutting boundary guide.
Architecture: Read this before diving into any reference repo. It tells your AI session what to
  reuse as-is vs. what to adapt.
Invocation: n/a (reference doc)
-->

# Generic vs. Lee-Specific — What to Reuse and What to Replace

The reference repos document one specific boat (SV Unladen Swallow 2, a Hylas 51). This
guide tells your AI session which parts are **transferable patterns** (use them as-is)
and which are **Lee's configuration** (replace them with your own).

## The rule of thumb

- **Architecture, methodology, and data formats** → generic, reuse them
- **Specific devices, addresses, thresholds, and pin assignments** → Lee-specific, replace them
- **Design decisions with reasoning** → generic (the reasoning transfers even if the choice doesn't)
- **Requirements labeled LEE** → Lee's preferences (read them, then write your own)

## Per-repo guide

### automation-engine

| File / section | Generic or Lee-Specific | How to use it |
|---|---|---|
| `spec/00-Requirements.md` | **Lee-specific** (his 19 requirements) | Read for inspiration. Write YOUR requirements using the same R-/N-/DR- format. Many of Lee's non-functional requirements (N-01 manual override, N-02 offline-first, N-06 advise-by-default) are good engineering for any boat. |
| `spec/01-Architecture.md` — the layered design | **Generic** (the PATTERN) | The layers (Pi-free hardware → ingest → logic → history → present → egress) are universal. Reuse the architecture; replace the components table with your equipment. |
| `spec/01-Architecture.md` — components table, addresses | **Lee-specific** | Replace with your devices and addresses. |
| `spec/04-Interfaces/rules-schema.md` | **Generic** (the rules format) | This is the rules language. Use it exactly as documented — it works on any installation. |
| `spec/04-Interfaces/io_master.yml` | **Lee-specific** (every pin assignment) | Write your own io_master.yml with your modules and channels. The STRUCTURE (module → channel → function → state) is generic. |
| `spec/04-Interfaces/devices.md` | **Lee-specific** (Lee's device register) | Create your own from your boat survey. The COLUMNS (model, location, status, D-/F- ids) are generic. |
| `spec/08-Workflows/` | **Mixed** | The CATALOG STRUCTURE (WF-nnn, level, severity, modes) is generic. The specific workflows (WF-107 anchor drag on S2-DI4) are Lee's. Your AI writes new workflows for your sensors. |
| `spec/11-Build-and-Restore.md` | **Mixed** | The PROCEDURE (Pi setup, Ansible deploy, secrets management, backup) is mostly generic. The specific hostnames, IP addresses, and stack versions are Lee's. |
| `spec/12-Operations.md` | **Mixed** | The PATTERNS (health watchers, test-a-rule, promote/demote, failure modes) are generic. The specific thresholds and circuit references are Lee's. |
| `spec/02-Decisions.md`, `03-Findings.md` | **Lee-specific** (his discoveries) but **the METHOD is generic** | Read them to understand what kinds of things get recorded. Start your own D-001 / F-001. |
| `rules.yml`, `signals.yml` | **Lee-specific** | Write your own. The FORMAT is generic (rules-schema.md). |
| `engine.yml` | **Lee-specific** | Write your own engine configuration. |

### vessel-ops

| File / section | Generic or Lee-Specific | How to use it |
|---|---|---|
| The code itself | **Generic** | The app runs on any boat. Schema, API, frontend are boat-independent. |
| `vessel_local.db` (the database) | **Lee-specific** | Start with an empty database (the app creates one on first run). |
| Owner Settings (in the app) | **Lee-specific** | Configure for your boat name, units, timezone. |
| `wiki/` (the engineering wiki) | **Generic** | Documents how the code works — applies to any installation. |
| The release package | **Generic** | Privacy-scrubbed, ready for another boat. |
| `backend/boat_link.py` integrations | **Lee-specific** (his Pi address, his API tokens) | Configure the env vars for YOUR automation stack, or leave them unset (the app works standalone). |

### routing

| File / section | Generic or Lee-Specific | How to use it |
|---|---|---|
| The pipeline code | **Generic** | Works for any vessel with a GPX route. |
| `config/vessel.toml` | **Lee-specific** | Fill in YOUR vessel parameters (hull speed, fuel burn, tank capacity). |
| `config/crew.toml` | **Lee-specific** | Fill in YOUR crew. |
| `config/baseline.pol` | **Lee-specific** | Get or generate a polar file for YOUR boat. |
| `data/arrival_places.toml` | **Lee-specific** (Caribbean destinations) | Extend or replace for your cruising grounds. |
| `wiki/` | **Generic** | Documents how the system works. |

### anchor-alarm

| File / section | Generic or Lee-Specific | How to use it |
|---|---|---|
| `reference/` (Python anchor algorithm) | **Generic** | The circle-fit algorithm works on any boat. |
| `main/` (ESP32 firmware) | **Mixed** | The firmware runs on the Waveshare ESP32-S3 4.3B specifically. Core algorithm is generic; hardware abstraction layer is board-specific. |
| `SPEC.md` sections 1-6 (anchor watch) | **Generic** | The anchor watch design (circle fit, rode floor, coarse guard) is universal. |
| `SPEC.md` sections 7-17 (Ship's Automation integration) | **Lee-specific** | These features (night watch, engine alerts, tank fill, hot water, cold box) depend on Lee's automation stack. A standalone anchor alarm uses sections 1-6 only. |
| `wiki/Hardware.md` | **Lee-specific** (one board) | Use as a reference; adapt for your hardware. |

### alarm-panel

| File / section | Generic or Lee-Specific | How to use it |
|---|---|---|
| The circuit pattern (pull-to-ground NPN + LED) | **Generic** | Each channel follows the same pattern — scale to any channel count. |
| `registers/01-master-al-to-faceplate.md` | **Lee-specific** (his 29 channels) | Design your own channel map for your alarm set. |
| `registers/05-bom-preliminary.md` | **Mixed** | Passive components (resistors, transistors, LEDs) are generic. Board dimensions and channel count are Lee-specific. |
| The EasyEDA project | **Lee-specific** | Use as a starting point; modify for your channel count and layout. |

### wiring-io

| File / section | Generic or Lee-Specific | How to use it |
|---|---|---|
| `wiki/Architecture.md` — principles 1-6 | **Generic** | These are good engineering principles for any alarm/automation system. |
| `wiki/Architecture.md` — Modbus map, placement | **Lee-specific** | Design your own module layout and addresses. |
| `wiki/Wiring-Sheets.md` — the drawing system | **Generic** (the SYSTEM) | The YAML → validated HTML wiring sheet approach is reusable. The specific sheets are Lee's. |
| `Diagrams/sheets/*.yml` | **Lee-specific** | Write your own sheets using the same YAML format. |
| `wiki/Circuit-Register.md` | **Lee-specific** (Lee's circuits) but **the FORMAT is generic** | Create your own register with the same columns (sheet, status, proof, what's left). |
| The commissioning pattern | **Generic** | Ring out → prove → record → accept → live. Use it for every circuit you build. |

### music-player

| File / section | Generic or Lee-Specific | How to use it |
|---|---|---|
| The app code | **Mostly generic** | Works on any Android phone with any Bluetooth stereo. |
| Bluetooth MAC address | **Lee-specific** | Change `BluetoothAutoPlay.kt` to your stereo's address. |
