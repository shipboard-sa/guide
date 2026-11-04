<!--
Purpose: Reference BOM with alternatives and budget tiers — Step 4 of the AI workflow.
Version: v3 (2026-11-04) — full DRAFT; v3: HAT + S3 + S5 as minimum Modbus set (Lee's correction).
Scope: PUBLIC — hardware shopping guide organized by tier.
Architecture: Generated from the gap analysis (03). References devices across all physical repos.
Invocation: n/a (reference doc)
-->

# Step 4 — Reference BOM

A shopping list organized by [architecture tier](02-Architecture-Tiers.md). Each tier adds
components to the previous one. Prices are approximate (2026) and shift — your AI session
can look up current prices.

**How to use this page:** run your [gap analysis](03-Gap-Analysis-Template.md) first. You
probably already own some of these items (an N2K bus, a Victron Cerbo, chart plotters). Your
AI session subtracts what you have and builds a BOM for what's missing.

---

## Tier 1: Dashboard monitoring (~$350-500)

Everything you need to see your boat's data on dashboards and record history.

| Category | Reference part | Alternatives | ~Price | Notes |
|---|---|---|---|---|
| **Computer** | Raspberry Pi 5 (8 GB) | Pi 5 4 GB (lighter load); Pi 4B 4 GB (cheaper, slower) | $80-95 | 8 GB recommended for SignalK + Grafana + InfluxDB together |
| **Storage** | NVMe SSD 256 GB + Pimoroni NVMe Base | USB SSD (slower); high-endurance SD card (last resort — SD cards fail on boats) | $25-50 | NVMe strongly preferred — SD cards corrode in marine environments |
| **Case** | Argon ONE M.2 or Pimoroni aluminum | Any vented case; bare board in a project box with standoffs | $15-25 | Needs passive cooling — no fan (salt air kills fans) |
| **N2K gateway** | Yacht Devices YDEN-02 (Ethernet) | Actisense NGW-1-STNG or NGW-1-ISO (USB); Yacht Devices YDNU-02 (USB, cheaper) | $190-280 | The YDEN-02 is Ethernet (one fewer USB device); the YDNU-02 is USB and ~$60 less. Actisense is the industry standard but USB-only |
| **Network** | Ethernet cable (shielded Cat6) + unmanaged switch | USB Ethernet adapter (if your Pi has no spare Ethernet) | $15-30 | The Pi needs a wired connection to the ship's LAN. Wi-Fi is unreliable in an aluminum hull |
| **Power** | 24V-to-5V buck converter (Pololu 5V 5A D24V50F5) | 12V-to-5V (if 12V boat); PoE splitter (if your switch does PoE) | $10-25 | Match your house bank voltage. USB-C PD adapters work from shore power |
| **NMEA 0183 adapter(s)** | USB-to-RS422 isolated adapter (e.g., Actisense USG-2, or FTDI USB-RS422 cable) | dAISy AIS receiver (USB, receives AIS only — no transmit); NMEA 0183 multiplexer (combines multiple talkers onto one USB port) | $25-80 ea | **One per 0183 talker** that isn't on N2K. Most VHF radios and AIS transponders output 0183 (not N2K) — you need a USB-serial adapter to get that data into SignalK. Budget 1-2 adapters. An isolated adapter is worth the extra cost (see [Glossary: isolated vs non-isolated](Glossary.md#isolated-vs-non-isolated-why-it-matters)). USB-direct-to-SignalK is more reliable than 0183-to-N2K gateway converters (see note below) |
| **Victron monitoring** | Victron Cerbo GX | Victron Venus GX (older, still works); GX Touch 50 (Cerbo + screen) | $290-400 | If you have Victron equipment, you probably already have one. Not required for Tier 1 but recommended — it feeds battery, solar, and inverter data to SignalK via MQTT |

**Tier 1 total (new, no Victron):** ~$400  
**Tier 1 total (new, with Cerbo):** ~$700  
**If you already have a Cerbo and N2K bus:** ~$130-200 (Pi + storage + power + cable + 0183 adapters if needed)

### Don't forget your 0183 devices

Your VHF radio and AIS transponder almost certainly output NMEA 0183, not NMEA 2000.
"Just buy the N2K version" is usually not a real option:

- **VHF:** N2K-native VHFs exist but are rare and significantly more expensive.
  The reference build specifically searched for one and it was a deliberate choice —
  most owners will have a 0183-only VHF.
- **AIS:** most recreational boats carry a Class B transponder (em-trak, Vesper,
  and similar — $300-800). These are almost universally 0183-only output. N2K-native
  AIS exists but is significantly more expensive, and Class A (which is more likely
  to have N2K) is $1,500+ and designed for commercial vessels — overkill for a
  cruising boat.
- **0183-to-N2K gateway converters** (devices that convert 0183 sentences to N2K PGNs
  on the backbone) are available from Yacht Devices and others, but reliability has
  been mixed. The reference build's experience with Yacht Devices converters on a
  B&G system: they all failed. A USB-to-RS422 adapter going straight to SignalK is
  simpler and more reliable — fewer failure points, no protocol translation.

**The practical path:** accept that 0183 is here to stay, buy USB-to-RS422 isolated
adapters, and let SignalK handle both protocols. One adapter per 0183 talker, or a
multiplexer if you have several.

**Check before you buy:** look at the back of your AIS and VHF. If you see small
screw terminals or a round plug labeled "NMEA" or "0183" with two wires (Tx+/Tx−),
that's 0183. If it has a standard DeviceNet/Micro-C round connector on a drop cable,
that's N2K (already on your backbone, no adapter needed).

See the [Glossary](Glossary.md#nmea-0183-vs-nmea-2000) for a full comparison of the
two standards.

### Software (free)

| Component | What it does |
|---|---|
| SignalK | Reads N2K + NMEA 0183 + MQTT + Modbus; central data hub |
| Grafana | Dashboards — design your own or use community templates |
| InfluxDB | Time-series history (yesterday's battery curve, last month's fuel) |

---

## Tier 2: Monitoring + alerts + basic protection (~$300-500 above Tier 1)

Adds Modbus I/O for sensors the N2K bus doesn't carry, a rules engine for alerts and
notifications, and — because the entry-level module has both DIs and relays — basic
protection capability from the start.

The Pi talks RS-485 directly through a HAT — no separate gateway needed.

**The minimum system: 8 relays + 8 DI + 8 AI.** Two Waveshare modules (a Relay D and an
Analog Input) plus the HAT give you a foundation to start building on — enough to monitor
sensors, drive protection relays, and read analog signals. Everything else scales from here.

| Category | Reference part | Alternatives | ~Price | Notes |
|---|---|---|---|---|
| **RS-485 / CAN HAT** | Waveshare RS485 CAN HAT for Pi | Waveshare USB TO RS485 adapter (no soldering, but one more USB device) | $15-20 | Mounts on the Pi GPIO header. Makes the Pi the Modbus RTU master directly — no separate Ethernet gateway needed at this tier |
| **Relay + DI module** | Waveshare Modbus RTU Relay (D) 8ch + 8 DI | Separate Waveshare Relay 8ch + IO 8CH (two boxes); any Modbus RTU relay+DI module | $35-50 | The Relay (D) gives you 8 relays (1NO+1NC, 10A) AND 8 isolated digital inputs in one module. NPN pull-to-ground DIs, COM on +24V. This is the minimum — one box handles both alarm inputs and protection outputs |
| **Analog input module** | Waveshare Industrial 8-Ch Analog Acquisition Module (Modbus RTU, 12-bit) | Any Modbus RTU analog input module (0-5V or 4-20mA) | $30-45 | For current sensing (pump runtime via Hall effect sensors), tank levels, or any analog sensor. 8 channels, 7-36V supply |
| **RS-485 cable** | Shielded twisted pair (Belden 9841 or equivalent) | Any shielded 2-conductor cable rated for data | $0.50-1/ft | Run in its own conduit, away from power cables. One run from the Pi to the last module on the chain |
| **120-ohm terminator** | Yageo 120R 1/4W (at the far end of the bus) | Any 120R resistor; some modules have a built-in switch | $0.10 | One at each end of the bus — the HAT provides one end, put the resistor at the last module |
| **Temperature sensors** | RuuviTag Pro (Bluetooth, via Cerbo) | DS18B20 1-Wire (wired, via Pi GPIO); any Modbus RTU temp probe | $30-40 ea | RuuviTags are wireless and battery-powered — easy to place in the fridge, freezer, engine room. The Cerbo collects them. Budget 3-6 sensors |
| **DIN rail** | 35mm top-hat DIN rail + end stops | Any standard DIN rail | $10-15 | For mounting the Modbus modules. Plan your layout before drilling |
| **Fuse block** | Blue Sea ST Blade fuse block (6 or 12 circuit) | Any marine-rated fuse block for the automation supply | $40-80 | Every new circuit needs overcurrent protection at the source |

**Tier 2 add (typical):** ~$300-450 (HAT, Relay D, Analog Input, cable, 4 RuuviTags, fuse block)  
**Running total (Tier 1 + 2, no Victron):** ~$650-950

### What you can do at Tier 2

- Bilge pump monitoring (float switch → Modbus DI → rule → phone notification)
- Temperature alerts (fridge too warm, engine room too hot)
- Shore power loss notification
- Battery state alerts (low voltage, high temperature)
- Current sensing for pump runtime monitoring (Analog Input + Hall effect donuts)
- Basic protection relays (engine kill, load shedding) — the Relay D has 8 channels
- Any "if X then notify me" or "if X then actuate Y" rule your sensors can detect

---

### Waveshare module comparison

Your AI session uses this table to pick modules for your build:

| Module | Relays | DI | DO | AI | Form factor | Role |
|---|---|---|---|---|---|---|
| **Relay (D) 8ch+8DI** | 8 (1NO+1NC, 10A) | 8 (isolated) | — | — | larger | Foundation: relays + inputs in one box |
| **IO 8CH** | — | 8 (NPN, pull-to-ground) | 8 (open-drain, 500 mA) | — | **smaller** | Scaling: more DIs + low-current DOs, less rail space |
| **Relay 8ch** | 8 (1NO+1NC, 10A) | — | — | — | medium | Relay-only: when you need more relays but no DIs |
| **Analog Input 8CH** | — | — | — | 8 (12-bit, 0-5V) | medium | Current sensing, tank levels, analog sensors |
| **Relay 30ch** | 30 (1NO, 10A) | — | — | — | large | Cost-effective bulk relays for large builds |

All modules are Modbus RTU, 9600 8N1, 7-36V supply (except Relay 8ch: 5V). Each gets
its own address on the bus. Chain them with shielded twisted pair, 120R at each end.

---

## Tier 3: Scaled protection + alarm panel (~$600-1,200 above Tier 2)

Scales up the IO layer (more modules, redundancy), adds an Ethernet gateway for bus
reliability, a physical alarm panel, and the full wiring infrastructure.

| Category | Reference part | Alternatives | ~Price | Notes |
|---|---|---|---|---|
| **Ethernet gateway** | Waveshare RS485 TO POE ETH (B) | Keep using the HAT alone (works fine for smaller installations) | $25-40 | Upgrades the bus: the Pi polls via Ethernet (tcp://gateway-ip), the HAT becomes the cold spare. PoE powered, built-in 120R terminator. Nice to have, not required — the HAT handles the bus directly |
| **Second relay module** | Waveshare Modbus RTU Relay 8ch (1NO+1NC per ch, 10A) | A second Relay (D); Waveshare 30-Ch Modbus RTU Relay (for large builds — see note below); any Modbus RTU relay module | $30-45 | The reference build puts engine kills on DIFFERENT modules (S3 and S6) so one module failure can't take out both. Non-latching — relays drop on power loss |
| **IO modules** | Waveshare Modbus RTU IO 8CH (8 DI + 8 DO) x 1-2 | Dedicated DI modules; use the Relay D's built-in DIs if 8 is enough | $25-35 ea | 8 digital inputs AND 8 digital outputs (open-drain, 500 mA sink-to-ground) in a smaller form factor than the Relay D. The DOs are low-current outputs — good for LED drivers, logic signals, or small solenoids. Takes less DIN rail space than a Relay D or Relay 8ch, making them the scaling module when you need more DIs |
| **Interposing relays** | Finder 55.33 3PDT (24V coil, DIN rail socket) | Finder 55.32 2PDT (fewer poles); any relay with appropriate coil voltage and contact rating | $10-20 ea | For signal inversion, contact multiplication, or interfacing with existing alarm hardware. The reference build uses about 12 interposing relays |
| **Terminal blocks** | Weidmuller/Phoenix feed-through, 2.5mm² | Wago 221 lever nuts (quicker but more expensive); any screw or spring terminal rated for the wire gauge | $2-5 ea | Ground blocks (see below), feed-through for signal distribution. Budget 20-40 |
| **Ground blocks** | Phoenix PTFIX 6/18X2.5 (18-port ground distribution) | Any ground distribution block; copper bus bar | $15-25 ea | The reference build uses 4 blocks for ground consolidation at the nav station and engine room |
| **Alarm panel** | Custom PCB (29 channels, LED indicators) | Commercial alarm panel (e.g., BEP CZone); individual LED indicators on a custom faceplate; no panel (software-only alarms) | $50-200 | The reference build's PCB is 29 channels with backlit labels, ordered from JLCPCB (~$50 for 5 boards + SMT assembly). A commercial marine alarm panel costs $300-800+ but has fewer channels and less flexibility |
| **Connectors** | KF142V-5.08 screw terminals (8-pin) | Wago 250 series; Molex KK; any screw terminal with appropriate pitch | $2-4 ea | For the alarm panel and module wiring. Budget 10-15 connectors |
| **Wire** | Ancor marine-grade tinned copper (18 AWG for signals, 14 AWG for relay loads) | Any ABYC TE-4 rated marine wire — must be tinned copper, not bare | $0.30-0.80/ft | Budget 200-400 feet total depending on boat size and run lengths |
| **Ferrules** | Bootlace ferrules (assorted kit, 0.5-2.5mm²) | Bare wire into screw terminals (works but not recommended) | $15-25 kit | For every stranded wire into a screw terminal. A kit with 800+ ferrules and the crimper is about $25-40 |
| **Heat shrink** | Adhesive-lined marine heat shrink (assorted) | Standard heat shrink (not for wet locations) | $15-25 kit | Adhesive-lined for any splice or connection that might see moisture |
| **Labels** | Brother P-touch TZe tape | Any label maker with durable tape; hand-written shrink labels | $30-50 | Label EVERY wire at both ends with its W-number |
| **Crimp tools** | Ratcheting crimp tool (insulated terminals) + ferrule crimper | Combined tool (compromise) | $30-60 | Don't use pliers. A proper ratcheting crimper makes reliable connections |

**Tier 3 add (typical):** ~$600-1,000  
**Running total (Tier 1 + 2 + 3):** ~$1,250-1,950

### Scaling note: the Waveshare 30-Channel Relay

If you're planning a large build (20+ relay channels), look at the Waveshare Modbus RTU
30-Ch Relay Module. It puts 30 relays in one enclosure at a lower per-channel cost than
buying four separate 8-channel modules. Trade-off: all 30 channels are on one module, so
you lose the redundancy of splitting kills across separate hardware. For a large build,
consider a 30-channel for general automation and a separate 8-channel for safety-critical
kills (engine stop, charge disconnect).

### What Tier 3 adds over Tier 2

- Bus redundancy (Ethernet gateway as primary, HAT as cold spare)
- Redundant engine kills on separate modules
- Physical alarm panel with LED indicators for every monitored condition
- More DI + DO channels (IO 8CH modules: smaller footprint, low-current digital outputs)
- Full wiring infrastructure (ground consolidation, labeled runs, fuse protection)
- Manual override switches that bypass automation (copper always wins)

---

## Tier 4: Full system (~$1,500-3,000 above Tier 3)

The complete system: maintenance tracking, routing, anchor watch, cockpit displays.

| Category | Reference part | Alternatives | ~Price | Notes |
|---|---|---|---|---|
| **Ship's PC** | Any mini PC (Intel NUC, Beelink, etc.) | A laptop dedicated to the boat; a second Pi (less capable) | $200-500 | Runs mx-inv (maintenance/inventory), hosts the routing pipeline, serves as backup automation host. Doesn't need to be powerful |
| **Analog input module** | Waveshare Industrial 8-Ch Analog Acquisition Module (Modbus RTU, 12-bit) | Any Modbus RTU analog input module | $30-45 | For current sensing (pump runtime from Hall effect sensors), tank levels, or any 0-5V / 4-20mA sensor |
| **Current sensors** | YHDC HSTS016L 20A split-core Hall effect | Any split-core Hall sensor matched to your load current | $8-15 ea | Non-invasive pump monitoring — clamp around the wire, no circuit break needed |
| **Anchor alarm** | Waveshare ESP32-S3-Touch-LCD-4.3B | Any ESP32-S3 with a display; a phone app (less reliable) | $50-70 | The reference build's standalone anchor alarm with circle-fit algorithm, touchscreen, and panel output. Powered from the N2K bus |
| **Cockpit display** | KIP (on SignalK, browser-based) | Grafana dashboards on a tablet; a dedicated chart plotter screen | $0-300 | KIP is free software displaying SignalK data. Run it on any browser — a weatherproof tablet, a repurposed chart plotter, or the helm phone |
| **Security cameras** | Reolink Home Hub + Argus 4 Pro (solar, wireless) | Any IP camera system; Frigate NVR on the Ship's PC | $150-300 | Solar-powered wireless cameras avoid running wires on deck |

**Tier 4 add (typical):** ~$500-1,200  
**Running total (full system):** ~$2,000-3,200

---

## What you probably already have

Most boats with modern electronics already own some of these. Check before buying:

| If you have... | You DON'T need... | Savings |
|---|---|---|
| Victron Cerbo GX (or Venus GX) | A separate MQTT broker or battery monitor gateway | $290-400 |
| NMEA 2000 backbone with instruments | N2K bus hardware (just add a gateway) | $100-300 |
| Ethernet switch / router aboard | A separate network switch for the Pi | $20-50 |
| Victron SmartShunt | A separate battery monitor | $100-150 |
| Garmin / B&G / Raymarine chart plotter with N2K | An N2K wind / depth / speed source (they're already on the bus) | $200-600 |
| An existing Raspberry Pi | The Pi itself (check if it's a Pi 4 or 5 — Pi 3 is too slow) | $80-95 |

---

## Supplier guide

| Supplier | Best for | Notes |
|---|---|---|
| **Amazon** | Waveshare modules, Pi accessories, wire, tools, DIN rail, generic hardware | Fast delivery; check seller ratings for marine components |
| **Pimoroni / Adafruit / SparkFun** | Raspberry Pi, HATs, cases, power supplies | Better stock than Amazon for Pi-specific items; Pi-approved resellers |
| **West Marine / Defender** | Marine-rated wire, terminal blocks, fuse blocks, Blue Sea products | Premium prices but correct marine ratings (ABYC TE-4 wire, ignition-protected) |
| **Mouser / Digikey** | Specific ICs, connectors, passive components, current sensors | Industrial quality; good for exact part numbers |
| **LCSC** | PCB components (if ordering a custom alarm panel from JLCPCB) | Integrated with JLCPCB assembly; lowest prices for SMT components |
| **JLCPCB** | Custom PCB fabrication + SMT assembly | The reference build's alarm panel: ~$50 for 5 boards + assembly |
| **Yacht Devices** | YDEN-02, YDNU-02, YDRI-04, YDCC-04 (N2K accessories) | Direct from manufacturer (yachtd.com); also on Amazon |
| **Victron dealers** | Cerbo GX, SmartShunts, Orion converters, MPPT controllers | Buy through authorized Victron dealers for warranty support |

---

## Lead times and sourcing notes

| Item | Typical lead time | Notes |
|---|---|---|
| Raspberry Pi 5 | In stock (2026) | The shortage is over; multiple resellers have stock |
| Waveshare modules | 1-2 weeks (Amazon); 2-4 weeks (direct) | Amazon stock varies; order from Waveshare direct if Amazon is out |
| YDEN-02 / YDNU-02 | 1-3 weeks | Marine dealers stock them; direct from Yacht Devices if not |
| Victron Cerbo GX | Usually in stock at marine dealers | Check victronenergy.com dealer locator |
| Custom PCB (JLCPCB) | 2-3 weeks (standard) | Includes fabrication + SMT assembly + shipping. Express = 1 week at 2-3x cost |
| Marine wire (Ancor) | In stock (West Marine, Amazon) | Buy more than you think — an extra 20% avoids a second order |
| PTFIX ground blocks | 2-4 weeks (Mouser/Digikey) | Phoenix Contact industrial product; not stocked everywhere |

---

## The reference build's actual parts

For reference, these are the exact parts installed on SV Unladen Swallow 2. Your AI
session can look these up in the source repos for detailed specifications:

**Modbus chain (as built):** Waveshare RS485 TO POE ETH (B) gateway (192.168.50.10) →
S6 (Relay 8ch, addr 1) → S1 (IO 8CH, addr 4) → S2 (IO 8CH, addr 2) → S3 (Relay D 8ch+8DI,
addr 3). Bus: 9600 8N1, Yageo 120R terminator at S3. S5 (Analog Input 8CH) ordered. The Pi
also has a Waveshare RS485 CAN HAT (the original bus master before the Ethernet gateway was
added) — it is the cold spare. A new build can start with just the HAT and add the gateway later.

**Source files:**
- `automation-engine/spec/04-Interfaces/devices.md` — every device with model and location
- `automation-engine/spec/04-Interfaces/io_master.yml` — every IO channel assignment and state
- `wiring-io/wiki/IO-and-Relays.md` — module table, channel assignments, relay pool
- `wiring-io/wiki/Circuit-Register.md` — what is built / proven / live
- `alarm-panel/registers/05-bom-preliminary.md` — PCB component BOM with LCSC part numbers
- `anchor-alarm/wiki/Hardware.md` — anchor box hardware facts and pin assignments
