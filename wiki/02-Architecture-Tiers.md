<!--
Purpose: Map requirements profiles (from the Decision Tree) to concrete system architectures.
  This is Step 2 of the AI workflow.
Version: v1 (2026-11-04)
Scope: PUBLIC — architecture selection guide for new owners.
Architecture: References automation-engine spec/01-Architecture.md for the full (Tier 4) design.
  Each tier is a subset of the full build — the layers stack, so Tier 3 includes everything in
  Tiers 1 and 2.
Invocation: n/a (reference doc)
-->

# Step 2 — Architecture Tiers

Your [requirements profile](00-Decision-Tree.md) maps to one of these tiers. Each tier
includes everything below it — Tier 3 includes all of Tier 2, which includes all of
Tier 1. Start at the tier that matches your needs; you can always add the next tier later.

## Tier 1: Dashboard monitoring (~$200)

**You get:** a central hub that collects data from your existing instruments and shows it
on dashboards accessible from any phone, tablet, or laptop on your boat Wi-Fi.

**Architecture:**
```
Your existing instruments (NMEA 2000, Victron, etc.)
        │
        ▼
   Raspberry Pi 5  ── SignalK server (data hub)
        │                    │
        ▼                    ▼
   Grafana dashboards    History database
   (any browser)         (PostgreSQL + TimescaleDB)
```

**Hardware:**
- Raspberry Pi 5 (4 GB or 8 GB) + NVMe storage + case + power supply
- NMEA 2000 gateway (e.g., Yacht Devices YDEN-02) — connects the N2K bus to the Pi via USB
- Ethernet cable to your boat's router/switch (wired, not Wi-Fi)

**Software (all free, open source):**
- SignalK — the data hub that speaks NMEA 2000, NMEA 0183, Victron, and dozens of other protocols
- PostgreSQL + TimescaleDB — time-series history (see every sensor reading for the past year)
- Grafana — dashboards (voltage over time, temperature trends, tank levels)

**What it does NOT do:** no alerts, no automation, no physical alarm panel.

**Reference:** `automation-engine/spec/01-Architecture.md` layers "Ingest" and "History"
plus "Present" (Grafana). The full spec describes the Tier 4 build — for Tier 1, you use
only SignalK + history + Grafana and skip the rules engine, relays, and alarm panel.

**Victron shortcut:** if your entire power system is Victron, a Cerbo GX already does most
of Tier 1 (dashboards via VRM, local display on a GX Touch). The Pi adds SignalK (which
aggregates non-Victron data too) and long-term history that VRM doesn't retain. If you
have a Cerbo, the Pi reads it over MQTT — they complement each other.

---

## Tier 2: Monitoring + alerts (~$800)

**You get:** everything in Tier 1, plus the system sends you phone notifications when
something needs attention.

**Architecture adds:**
```
   Raspberry Pi 5 (from Tier 1)
        │
        ├── Rules engine (data-driven: YAML rules, not code)
        │       │
        │       ▼
        │   Pushover / ntfy (phone notifications)
        │
        └── (optional) Modbus I/O modules for additional sensors
```

**Additional hardware (beyond Tier 1):**
- Waveshare RS485 CAN HAT — sits on the Pi's GPIO header, makes the Pi the Modbus bus master directly
- Waveshare Modbus RTU Relay (D) 8ch + 8 DI — one module with 8 relays AND 8 digital inputs,
  enough for basic monitoring AND basic protection (engine kill, load shedding)
- Waveshare Analog Input 8CH — for current sensing (pump runtime via Hall effect) and analog sensors
- RS-485 cable (shielded twisted pair) — one run from the Pi to the modules
- Pushover app license ($5 one-time per phone)

The Relay (D) is the minimum Waveshare module: one box handles both alarm inputs and
protection outputs. An Ethernet gateway is a Tier 3 upgrade (bus redundancy, PoE) — the
HAT on the Pi handles the bus directly.

**What it adds:**
- Rules engine reads SignalK and Modbus data, evaluates conditions, sends alerts
- Rules are YAML data files, not code — your AI session writes them for you
- Phone notifications: bilge pump running too long, voltage high, anchor dragging,
  temperature out of range, AIS target too close
- Basic relay control (engine kill on cooling failure, load shedding on low battery)

**What it does NOT do:** no alarm panel (Tier 3), no bus redundancy (Tier 3).

**Reference:** `automation-engine/spec/01-Architecture.md` adds the "Logic" layer (rules
engine) and "Egress" layer (notifier). The rules format is in
`automation-engine/spec/04-Interfaces/rules-schema.md`. The [Rules Authoring Tutorial](06-Rules-Authoring-Tutorial.md)
teaches you how to write rules.

---

## Tier 3: Monitoring + alerts + protection (~$2,000)

**You get:** everything in Tier 2, plus the system can take protective action (with
manual override always available in copper), and a physical alarm panel provides backup
that works even when the Pi is off.

**Architecture adds:**
```
   Sensors ──┬──► Master Alarm Panel (standalone, Pi-free)
             │        │
             │        ▼ lamps, buzzers, manual mute
             │
             └──► Modbus I/O modules ──► Raspberry Pi
                       │                      │
                       ▼                      ▼
                  Relay pool             Rules engine
                  (automated actions)    (decides when to act)
```

**Additional hardware (beyond Tier 2):**
- Ethernet gateway (Waveshare RS485 TO POE ETH) — upgrades the bus; HAT becomes cold spare
- Second relay module on a different Modbus address — redundant engine kills on separate modules
- Additional IO 8CH module(s) — more digital inputs as your sensor count grows
- Physical alarm panel — LED indicators, buzzers, mute button (the reference build uses
  a custom PCB from `alarm-panel`; you can also use a commercial marine alarm panel)
- Parallel sensor taps — each sensor feeds BOTH the alarm panel AND the Modbus I/O, so
  the panel works independently of the Pi
- DIN rail, terminal blocks, wiring supplies — see [Before You Build](05-Before-You-Build.md)

**Design principles (from the reference build):**
1. **The alarm panel is primary and standalone.** If the Pi dies, you still have alarms.
2. **Ship's Automation is advisory.** Its failure is announced on the panel.
3. **Manual control stays in copper.** Every automated output has a physical override switch.
4. **Fail-safe direction is chosen per output.** Engine kills: NC in series (dead relay =
   engine runs). Charge disconnect: NC energise-to-disconnect (dead relay = loads stay on).

**What it adds:**
- Engine kill on cooling loss or fire
- Charging cutoff on overvoltage (battery protection layers)
- Blower control (automatic on engine start, stop on fire)
- Horn sounding on unacknowledged alarms
- Acknowledge/re-sound pattern for alarm management
- Physical alarm panel with battery backup

**Optional at this tier:**
- Standalone anchor alarm (anchor-alarm repo) — adds a dedicated GPS anchor watch with
  touchscreen, independent of the Pi
- Passage planning (passage-planner) — weather-informed routing and briefings

**Reference:** `automation-engine/spec/01-Architecture.md` — the full layered architecture.
`wiring-io/wiki/Architecture.md` — the physical wiring layer (CAMS 1.5 design).
`wiring-io/wiki/Circuit-Register.md` — every circuit with build status (your commissioning
register will follow this pattern).

---

## Tier 4: The full system (~$5,000+)

**You get:** the complete reference build as documented across all 7 repos.

**Architecture adds (beyond Tier 3):**
- Maintenance and inventory system (vessel-ops) — scheduling, spares, food, ship's log
- Cockpit display — helm phone/tablet showing watch info, checklists, alarms
- Passage planning (passage-planner) — GPX route in, weather-informed briefing out
- Ship's log — automatic deck log from engine events, manual entries, review workflow
- Custom alarm panel PCB (alarm-panel) — 29-channel LED panel with backlit labels
- Music player (music-player) — shuffle player for boat stereo

This is SV Unladen Swallow 2's complete system. It took ~6 months of evenings and weekends
with two people (one engineer doing vision/requirements/wiring, one AI doing software).
Most of that time was discovery — your build will be faster because the design already
exists.

**Reference:** every repo. Start with `automation-engine/spec/README.md` for the spec index,
then `vessel-ops/wiki/Home.md` for the maintenance system, `routing/wiki/Home.md` for
passage planning.

---

## Choosing your tier

| Your profile says | Start at |
|---|---|
| Monitor only / Weekender / ~$200 / Basic | Tier 1 |
| Monitor+Alert / Coastal / ~$800 / Basic-to-DC | Tier 2 |
| Monitor+Alert+Protect / Offshore / ~$2,000 / DC-comfortable | Tier 3 |
| Full system / Liveaboard / $5,000+ / Electrician or Coder | Tier 4 |

**You can always upgrade.** Each tier is a superset of the one below. Start with Tier 1,
use it for a season, and add Tier 2 when you're ready. The Pi and SignalK hub are the
foundation that everything else builds on.

## What your AI session does next

With your [requirements profile](00-Decision-Tree.md) and your tier selected:

1. **[Gap analysis](03-Gap-Analysis-Template.md)** — compare your boat survey to the
   tier's requirements, identify what hardware and wiring you need.
2. **[Reference BOM](04-Reference-BOM.md)** — your AI generates a shopping list filtered
   to your tier, with alternatives for your specific equipment.
3. **[Before You Build](05-Before-You-Build.md)** — if your tier includes wiring (Tier 2+),
   read the marine electrical fundamentals first.
