<!--
Purpose: Structured survey form for collecting what's already on a new owner's boat.
  This is Step 1 of the AI workflow — the input to architecture design and gap analysis.
Version: v1 (2026-11-04)
Scope: PUBLIC — boat-independent survey template.
Architecture: Produces a filled survey that feeds into 02-Architecture-Tiers and
  03-Gap-Analysis-Template. References automation-engine spec/04-Interfaces/devices.md
  for the worked example (Lee's completed survey).
Invocation: n/a (reference doc)
-->

# Step 1 — Boat Survey Template

Walk through each section with your AI session. Fill in what you know; mark unknowns as
**"? — check"**. Your AI can help identify equipment from photos, model numbers, or
manual excerpts. This survey feeds directly into the [architecture design](02-Architecture-Tiers.md)
and [gap analysis](03-Gap-Analysis-Template.md).

## 1. Vessel basics

| Item | Your boat | Notes |
|---|---|---|
| Make / model | | |
| Year | | |
| LOA / beam / draft | | |
| Hull material | | fiberglass / aluminum / steel / wood-epoxy |
| Rig type | | sloop / cutter / ketch / cat |
| Propulsion | | engine make/model, HP, fuel type |
| Generator | | make/model or "none" |
| Electrical system voltage | | 12V / 24V / 12+24V |
| Shore power | | 30A / 50A / none |

## 2. Navigation electronics

| Item | Make / model | Bus | Notes |
|---|---|---|---|
| Chartplotter 1 | | NMEA 2000 / Ethernet / standalone | |
| Chartplotter 2 | | | |
| VHF radio | | NMEA 2000 / NMEA 0183 / standalone | DSC? AIS Rx? |
| AIS transponder | | NMEA 2000 / NMEA 0183 / USB | Class A / Class B |
| Depth / speed / temp | | NMEA 2000 / NMEA 0183 | transducer model? |
| Wind instrument | | NMEA 2000 / NMEA 0183 | masthead / ultrasonic? |
| Autopilot | | NMEA 2000 / Seatalk / standalone | make/model, heading source |
| Radar | | Ethernet / standalone | |
| GPS antenna(s) | | NMEA 2000 / USB | how many, where mounted |
| Satellite comms | | | inReach / Iridium GO / Starlink / none |

## 3. Power system

| Item | Make / model | Details | Notes |
|---|---|---|---|
| House batteries | | chemistry, capacity (Ah), voltage, count | |
| Start battery | | chemistry, capacity | |
| Battery monitor / BMS | | make/model, data interface | NMEA 2000 / Bluetooth / CAN / Victron VE.Bus / none |
| Shore charger / inverter | | make/model, capacity | Victron / Magnum / Xantrex / other |
| Solar panels | | total watts, controller make/model | MPPT? data interface? |
| Alternator | | stock / high-output, regulator | external regulator? data? |
| Wind generator | | make/model or "none" | |
| DC-DC converter | | make/model or "none" | 24→12V / 12→24V |
| Battery cutoff / protect | | make/model or "none" | BatteryProtect / manual switch |

### Power system data interfaces

This is critical for architecture design. For each power component, note:

| Component | Data interface | Protocol | Notes |
|---|---|---|---|
| Battery monitor / BMS | | VE.Direct / VE.Bus / CAN / Bluetooth / Modbus / none | |
| Inverter / charger | | VE.Bus / VE.Direct / CAN / Modbus / none | |
| Solar controller(s) | | VE.Direct / Bluetooth / none | |
| Cerbo GX / Venus OS | | MQTT / Modbus TCP / D-Bus / "don't have one" | if Victron, a Cerbo or similar GX device is the data hub |

## 4. Alarm and safety systems

| Item | Make / model | Notes |
|---|---|---|
| Existing alarm panel | | make/model, channel count, or "none" |
| Smoke detectors | | count, location, type (ionization / photoelectric) |
| CO detector | | make/model or "none" |
| Fire suppression | | Fireboy / other / none; auto or manual |
| Bilge pumps | | count, locations, float switches? |
| Bilge alarms | | high-water floats? count? |
| Engine alarms | | what's wired: oil pressure, coolant temp, raw water? |
| Propane / LPG | | solenoid? sniffer? or "all-electric galley" |

## 5. Bus infrastructure

What data networks exist on your boat today?

| Bus | Present? | Details |
|---|---|---|
| NMEA 2000 backbone | yes / no | length, number of drops, T-connectors available |
| NMEA 0183 | yes / no | which devices, baud rate if known |
| Seatalk (original) | yes / no | Raymarine legacy |
| Seatalk NG | yes / no | Raymarine, compatible with NMEA 2000 |
| Ethernet | yes / no | switch? router? what's connected? |
| Wi-Fi | yes / no | router make/model, coverage |
| Cellular | yes / no | router / hotspot / phone tethering |
| USB | yes / no | USB-to-serial adapters in use? |

## 6. Physical layout

Sketch or describe. Your AI session needs this to plan wire runs and equipment placement.

| Item | Description |
|---|---|
| Nav station location | |
| Nav station panel space | approximate dimensions available for new equipment |
| Engine room access | easy / tight / terrible |
| Battery compartment location | |
| Electrical panel location | |
| Wire run: nav station → engine room | approximate distance, existing conduit? |
| Wire run: nav station → battery | approximate distance |
| DIN rail space available | existing DIN rail? space to add? |
| 24V/12V distribution | breaker panel? fuse block? bus bars? |

## 7. What's already monitored or automated?

| Function | How it works today | Want to keep / replace / add? |
|---|---|---|
| Engine alarms | | |
| Battery voltage | | |
| Bilge monitoring | | |
| Anchor alarm | | chartplotter / standalone / none |
| Temperature monitoring | | fridge, freezer, cabin? |
| Security / intrusion | | |
| Lighting automation | | |
| Tank levels | | |

## Worked example

The completed survey for SV Unladen Swallow 2 (Hylas 51) is effectively the content of:
- `automation-engine/spec/04-Interfaces/devices.md` — every device, location, status
- `automation-engine/spec/01-Architecture.md` — ingest ownership table (what talks to what)
- `wiring-io/wiki/Panel-Points.md` — the 21-point alarm panel as-built
- `wiring-io/wiki/IO-and-Relays.md` — Modbus modules and channels

These files show what a COMPLETED survey looks like. Your survey will be shorter (you're
starting, not documenting a finished build). The AI session uses your survey answers to
pick which reference repo content applies to your boat.
