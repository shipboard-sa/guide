<!--
Purpose: Marine electrical fundamentals — the "Before You Build" hardware guide for
  Tier 2+ builds. Step 5 of the AI workflow.
Version: v2 (2026-11-04) — DRAFT. Needs Lee's review for technical accuracy.
Scope: PUBLIC — safety-critical content, verified against ABYC E-11 / ISO 10133 principles.
Architecture: Read before any physical wiring work. References wiring-io for the worked example.
Invocation: n/a (reference doc)
-->

# Step 5 — Before You Build

This page covers the marine electrical fundamentals you need BEFORE running any wire.
If you're at Tier 1 (Pi only, no wiring beyond plugging in cables), you can skip this.
Tier 2+ involves wiring sensors and relays into your boat's electrical system — get this
right or don't do it.

---

## Start here: Victron's Wiring Unlimited

Before reading this page, read **Victron's "Wiring Unlimited"** (free PDF from
victronenergy.com). It covers:

- Ohm's law, cable resistance, and voltage drop calculations
- Why cable thickness and length matter (with worked examples)
- Battery bank wiring (series, parallel, balancing)
- DC distribution: busbars, fuse holders, isolation switches
- Cable connections: lugs, ferrules, crimp terminals, torque
- Fuse selection: current rating, voltage rating, speed (slow-blow for DC), type
- AC wiring, circuit breakers, and RCDs
- Grounding in mobile installations (boats and vehicles)
- Galvanic corrosion and galvanic isolators

Wiring Unlimited is 69 pages of well-illustrated, practical guidance. It is not
boat-specific in every section, but its Chapter 7.5 (Mobile Installations) and Chapter 8
(Galvanic Corrosion) are directly about boats.

**This page does NOT repeat what Wiring Unlimited covers.** Instead, it supplements
Wiring Unlimited with the things specific to automation wiring on a boat: low-voltage
signal runs, DIN rail layout for Modbus modules, RS-485 bus wiring, labeling, and the
tools and safety practices for this particular kind of work.

### One thing Wiring Unlimited covers that you must not skim

**Batteries wired in parallel must have identical-length cables.** Not approximately
the same — identical. If one parallel path is shorter than another, it has lower
resistance, so it carries more current, charges faster, discharges faster, and ages
faster than its neighbors. Over time, the bank becomes unbalanced: the short-cable
battery does all the work, wears out first, and drags down the bank.

Measure, cut to the same length, and verify with a meter. If the physical layout makes
equal lengths awkward, add length to the shorter run — a little extra cable is far
cheaper than an unbalanced bank.

This is one of the strongest arguments for busbars (see the battery bank interconnects
section below): a busbar between adjacent posts is the same length by definition.
No measuring, no matching — the equal-resistance problem simply disappears.

---

## 1. What Wiring Unlimited doesn't cover

Wiring Unlimited focuses on high-current DC power paths (battery → inverter → loads)
and AC distribution. Your automation system adds a different kind of wiring:

| Wiring Unlimited covers | This page covers |
|---|---|
| Battery cables (2/0 AWG, 200+ A) | Signal cables (18-22 AWG, milliamps) |
| Power fusing (MEGA, ANL, MCB) | DIN fused terminal blocks (2-5 A) |
| Cable lugs and busbar connections | Ferrules into screw terminals |
| AC circuit breakers | RS-485 bus termination |
| Power distribution panels | DIN rail automation enclosures |

Both types of wiring live on the same boat and share the same grounding system. The
fundamentals (voltage drop, connection quality, fuse-at-source, tinned copper) apply to
both — Wiring Unlimited teaches those. This page teaches the rest.

---

## 2. Plan for growth

This is likely new territory for you. You've read what the reference build did and
thought "cool — I'd like some of that." But your boat is different, your needs are
different, and here's the thing: **your understanding of what's possible will grow
faster than you expect.**

You'll start building, and as you gain confidence and understanding, the envelope
describing the art of the possible expands. Ideas you didn't have at the start become
obvious once you see the system working. A $20 sensor is easy to justify. A better
module that gives you more capability for less money is a no-brainer. The system grows
— and it should. That's the point.

**A real example from the reference build:** midway through commissioning, we swapped a
4-channel on/off relay input device for an 8-channel analog module with current sense —
more channels, better data, less money. Instead of just knowing "the bilge pump is on,"
we could tell whether the pump was running hard or running dry. Worth doing? Absolutely.
But that module needed DIN rail space we hadn't planned for, in the back of a cabinet
that's difficult to reach. Same story when we realized the Fusion stereo could serve as an alarm enunciator — the
radio is already on N2K, but keying the audio alert meant an extra relay we hadn't
planned for.

**The problem is infrastructure that can't grow with you.** The things that limit you
later are the things you sized "just right" at the start:

- **Network switch with no spare ports** — adding a device means replacing the switch
  and re-running Ethernet
- **Relay module with every channel used** — one more relay means a second module, a
  new DIN rail spot, new power and bus wiring
- **Enclosure that's full** — a bigger enclosure means moving everything
- **Ground consolidation block with no empty positions** — a new sensor needs a ground
  point you don't have
- **DIN rail with no empty space** — no room for the next module

**Buy one size up on everything that's hard to change later.** An 8-port switch instead
of 5. A 16-channel relay module instead of 8. An enclosure 30% bigger than your layout.
An 18-position PTFIX block instead of 12. The cost difference at purchase time is small.
The cost of replacing undersized infrastructure after it's wired, mounted, and labeled
is enormous.

**Leave 30-50% of every resource unused at commissioning.** If you're at 50-70% of
your DIN rail, switch ports, relay channels, or ground points on day one, you have room
for the ideas that come at month three — and they will come.

---

## 3. Signal wire: gauge and routing

### What gauge for what

| Application | Gauge | Why |
|---|---|---|
| RS-485 bus (Modbus RTU) | 22 AWG shielded twisted pair | Low current (~mA), needs shielding against EMI |
| Dry-contact sensor runs (float switches, reed switches) | 18-22 AWG stranded | Current is negligible; gauge is about mechanical durability |
| 4-20 mA analog sensor loops | 18-20 AWG shielded twisted pair | Loop current is small but noise rejection matters |
| Relay load wiring (pumps, solenoids, engine kill) | 14-16 AWG per load | Sized for the load current, not the signal |
| Module power supply (12/24V to Modbus modules) | 16-18 AWG | Each module draws ~0.5-1.5 A; size for the total bus load + distance |

### Voltage drop still matters at 12V

Wiring Unlimited's 2.5% rule applies to your module power supply wires too. At 12V,
2.5% is only 0.3V. A long, thin power run to a distant Modbus module can drop enough
voltage to cause unreliable operation or relay chatter.

**Rule of thumb for module power:** calculate the total current draw of all modules on
the run, then use Wiring Unlimited's voltage drop formula (V = I × R, where R includes
round-trip cable length) to verify you're under 0.3V drop at 12V or 0.6V at 24V.

### Routing rules

- **Keep signal cables away from power cables.** Run them in separate cable trays or on
  opposite sides of the conduit. Where they must cross, cross at 90 degrees.
- **Keep RS-485 away from inverters and AC wiring.** Inverters generate significant EMI.
  Minimum 12" (30 cm) separation, more if possible.
- **Use shielded cable for RS-485 and analog runs.** Ground the shield at ONE end only
  (the Pi/gateway end). Grounding both ends creates a ground loop that introduces the
  noise you're trying to reject.
- **Strain relief at every entry point.** Use cable glands or strain relief bushings where
  cables enter enclosures. A cable that can move will eventually break at the terminal.

---

## 4. Connections and terminals

Wiring Unlimited covers cable lugs, ferrules, and crimp terminals in its Chapter 4.3.
For automation wiring, the specifics are:

### Ferrules — use them everywhere

Every stranded wire that goes into a screw terminal gets a bootlace ferrule. No
exceptions. A bare stranded wire in a screw terminal will:

- Splay when tightened (some strands miss the clamp)
- Loosen over time from vibration
- Create a high-resistance joint that heats up

**Ferrule sizing:** Match the ferrule to the wire gauge. A 0.5 mm² ferrule for 22 AWG,
0.75 mm² for 20 AWG, 1.0 mm² for 18 AWG. Color-coded by size.

**Ferrule crimper:** Use a proper ferrule crimper (self-adjusting ratchet type, not
pliers). A bad crimp is worse than no ferrule — it creates a hidden failure point.

### Crimp types for automation wiring

| Terminal | When to use | Tool |
|---|---|---|
| Bootlace ferrule | Every stranded wire → screw terminal | Ferrule crimper (self-adjusting) |
| Ring terminal | Wire → stud or bolt (ground bus, battery terminal) | Ratcheting crimp tool |
| Spade terminal | Wire → barrier strip where you need to disconnect often | Ratcheting crimp tool |
| Butt splice | Inline wire join (use sparingly — a terminal block is better) | Ratcheting crimp tool |

**Always:** Tinned copper terminals. Adhesive-lined heat shrink over any crimp that
might see moisture (engine room, bilge area). Ratcheting crimper, never pliers.

### Battery bank interconnects: busbars vs cables

For the connections between batteries in your bank, consider **copper busbars** instead
of cables. The choice depends on your battery layout:

- **Batteries in a row (terminals line up):** Busbars bolt directly across adjacent
  posts — solid copper, no crimp joints to fail, lower resistance than any cable, and
  easy to visually inspect. Run a cable from the + terminal at one end and the −
  terminal at the opposite end to your distribution.
- **Batteries in multiple rows or columns:** Use busbars within each row where
  terminals line up, then cables to bridge between rows where the geometry doesn't
  allow a rigid bar.

Busbars eliminate the failure mode that causes the most trouble in battery banks:
high-resistance crimp joints that heat up under load. A bolted busbar connection is
inspectable (you can see and feel a loose bolt) and re-torqueable. A bad crimp inside
a cable lug is invisible until it melts.

**Never on a boat:** Wire nuts (vibration loosens them), push-in spring connectors in
high-vibration locations, solder-only joints on structural connections (vibration fatigue
and solder wicking make the wire brittle at the joint boundary).

---

## 5. DIN rail and enclosure layout

Your Modbus modules, terminal blocks, fuses, and power supplies mount on standard 35mm
DIN rail inside an enclosure. Plan the layout before you drill.

### Layout principles

1. **Power in at the top, signals out at the bottom.** Convention: power supply and main
   fuse at the top of the rail, modules in the middle, terminal blocks for field wiring at
   the bottom.

2. **Leave 30-50% empty rail for future modules.** You will add sensors. Plan for it now.

3. **One module width of air gap between modules.** Waveshare modules generate heat
   when all relays are energized. Don't pack them tight.

4. **Wire duct (Panduit) on both sides of the rail.** Keeps wiring tidy, protectable, and
   accessible. Size the duct for 50% fill — wires need room to bend and pull.

5. **End stops on every rail.** DIN rail end stops prevent modules from sliding. On a boat,
   everything slides.

6. **Cable ties rated for UV and vibration.** If any cable route sees sunlight, use UV-rated
   ties. In the engine room, use high-temp ties.

### Terminal blocks

| Type | Use |
|---|---|
| Feed-through | General signal distribution — one wire in, one wire out per terminal |
| Ground (green/yellow) | Dedicated ground bus on the DIN rail |
| Fused (with LED blown indicator) | Inline fuse holder on the DIN rail — one per branch circuit. **Use the LED-indicator variant** |
| PTFIX distribution block | Ground consolidation — multiple wires to a single ground point |

### LED fuses vs plain inline fuses

DIN fused terminal blocks come in two variants: plain (you pull the fuse to check it)
and LED-indicator (an LED lights when the fuse blows). **Always use the LED variant.**

On a panel with 15-20 fused circuits, a plain fuse forces you to pull and test each one
to find the dead circuit — in the dark, at sea, while the alarm is telling you something
is wrong. An LED-blown indicator makes the failed circuit immediately visible: walk up
to the panel, see the lit LED, replace that fuse.

The cost difference is negligible (a few cents per block). The diagnostic difference at
0300 in a seaway is not.

The reference build uses PTFIX 6/18X2,5 blocks at the nav station and engine room for
ground consolidation. Each block provides 18 connection points on a single DIN-mount
footprint.

### Enclosure selection

- **IP rating:** IP65 minimum for engine room or any location that might see spray. IP44
  is adequate for an interior nav station installation.
- **Material:** Fiberglass or polycarbonate (non-conductive). Metal enclosures work but
  must be bonded to the boat's grounding system.
- **Size:** Measure your planned layout, then go one size up. You will fill it.
- **Ventilation:** If the enclosure holds relay modules under continuous load, it needs
  ventilation (filtered vents or a small fan). Sealed boxes in a hot engine room will
  cook the electronics.

---

## 6. RS-485 bus wiring

The Modbus RTU bus that connects your Pi (or Ethernet gateway) to the I/O modules
runs on RS-485. This is a differential signal on a twisted pair — simple but intolerant
of bad wiring.

### Cable

Use shielded twisted pair, 22 AWG, rated for at least 300V insulation. Cat5e/Cat6
cable works in a pinch (use one twisted pair for A/B, leave the rest unused) but
purpose-made RS-485 cable (e.g., Belden 3105A or equivalent) has better shielding and
is easier to terminate.

### Topology: daisy chain

RS-485 is a bus, not a star. The cable runs from the master (HAT or gateway) to module
1, then from module 1 to module 2, and so on. Do NOT run separate cables from the
master to each module — that's a star topology and it creates signal reflections.

```
Pi/HAT ──── Module 1 ──── Module 2 ──── Module 3
 120Ω                                      120Ω
```

### Termination

A 120-ohm resistor at each end of the bus. The RS485 CAN HAT usually has a built-in
termination resistor (check the DIP switch or jumper — enable it). Put a second 120Ω at
the LAST module on the chain. No termination in the middle.

### Shield grounding

Ground the shield at the master end only. Leave it floating at every module. Grounding
both ends creates a ground loop.

### Common pitfalls

- **A/B swap.** Waveshare labels A and B consistently, but other manufacturers sometimes
  swap them. If a module doesn't respond, try swapping A and B at that module.
- **Baud rate mismatch.** The reference build uses 9600 8N1. Every module on the bus must
  match. Check each module's DIP switches or holding registers.
- **Stub length.** If a module can't sit directly on the bus, keep the stub (branch cable)
  under 1 meter. Long stubs cause reflections.
- **Bus length.** RS-485 is rated for 1200m, but keep it practical. On a boat, 30-50
  meters of total bus length is typical. At 9600 baud, length is rarely the limiting
  factor.

---

## 7. Grounding for automation systems

Wiring Unlimited's Chapter 7 covers grounding thoroughly, including mobile installations
(boats). Read it first. What follows is specific to the low-voltage automation layer.

### Three grounds on your boat — they are different

| Ground | What it is | Color | Where it goes |
|---|---|---|---|
| **Safety ground** (AC) | Fault return path for AC equipment | Green or green/yellow | Shore power → galvanic isolator → boat's bonding system |
| **DC negative** | Return path for the 12/24V DC system | Black (usually) | Battery negative bus → distribution → all DC loads |
| **Signal ground** | Reference for RS-485, sensors, module logic | Per module spec | Module GND terminal → local ground consolidation block |

**Do not assume these are the same wire.** On many boats they are connected at specific
points (the bonding system), but running a sensor's signal ground to the AC safety ground
bus will introduce noise.

### Signal ground strategy

For the automation modules:

1. Each module has a GND terminal. Connect it to a local PTFIX ground consolidation block.
2. The PTFIX block connects to the DC negative bus at ONE point (single-point ground for
   the automation system).
3. The RS-485 cable shield grounds at the master end only (see above).

This keeps the automation system's ground clean. Noise from the shore power, inverter,
or engine alternator stays on the power ground, not on your signal ground.

### Grounds, grounds, grounds

**Plan for at least twice as many ground conductors as power conductors.** This is the
single most underestimated part of any automation build.

Every sensor needs a ground return. Every Modbus module needs a ground. Every relay
common needs a ground. Every shielded cable has a shield drain wire that lands on a
ground point. Add them up and you will have two to three ground wires for every power
wire in the system.

What this means in practice:

- **Size your ground consolidation blocks generously.** If you think you need 12 ground
  points, buy an 18-position PTFIX block. You will fill it.
- **Run dedicated ground conductors.** Don't daisy-chain grounds through modules —
  give each module its own home run to the consolidation block. A shared ground wire
  that develops a high-resistance joint takes down everything downstream.
- **Buy more green/yellow wire than you think.** When you estimate your wire order,
  double the ground conductor quantity relative to power. You will use it.
- **Leave empty positions on every ground bus and PTFIX block.** Future sensors and
  modules will need ground points. If the block is full, you're running a new block
  and a new home run — plan ahead.

Most wiring problems in automation systems are ground problems. A floating ground, a
shared ground with too much current, or a missing ground return will cause intermittent
sensor readings, Modbus communication errors, and false alarms that are maddeningly
hard to diagnose. Copper is cheap. Troubleshooting is not.

---

## 8. Labeling

Label every wire at both ends. No exceptions. A wire that you can trace today will be
a mystery in 6 months, especially inside a wire duct.

### Wire numbering scheme

The reference build uses a W-number system: W01-01 through W16-xx, where the first
number is the wiring sheet and the second is the wire within that sheet. Any consistent
scheme works — the point is that every wire has a unique ID that appears on:

- The wire itself (both ends)
- The wiring diagram
- The terminal block label

### Don't trust insulation color

On an existing boat, wire colors may not match any standard. Previous owners, boatyard
electricians, and factory wiring from different decades all used different conventions.
The reference build found wires where the color bore no relation to the function.

**Your labels are the system of record, not the wire color.** Label, then verify with a
multimeter.

### Label maker

A Brother P-touch (or equivalent) with TZe tape rated for the environment. Use white
tape with black text for most locations. Use yellow tape for safety-critical circuits.
Laminated tape survives better than non-laminated in marine environments.

---

## 9. Tools

The minimum tool kit for Tier 2+ wiring. **Where to buy:** Amazon, Lowe's, and
Home Depot carry everything on this list. These are general-purpose tools — there's
no reason to pay marine chandlery or specialty electronics prices for wire strippers
and crimpers.

| Tool | What for | Budget |
|---|---|---|
| Heavy-duty wire cutters | Cutting 10-2/0 AWG power cable cleanly | $20-40 |
| Flush-cut (zero-clearance) cutters | Trimming zip ties, small wire, ferrule tails — cuts flush against the surface | $10-25 |
| Wire strippers (22-10 AWG range) | Stripping signal and power wire | $15-30 |
| Self-adjusting ferrule crimper | Bootlace ferrules for screw terminals | $25-40 |
| Ratcheting crimp tool (insulated terminals) | Ring, spade, butt splice terminals | $30-50 |
| Digital multimeter | Continuity, DC voltage, resistance, AC voltage | $30-80 |
| Heat gun | Adhesive-lined heat shrink | $20-40 |
| Label maker (Brother P-touch or similar) | Wire and terminal block labels | $30-60 |
| Screwdriver set (including small flat for terminal screws) | Module terminal screws, DIN rail work | $15-25 |
| Needle-nose pliers | Wire routing, holding ferrules | $10-15 |

### Consumable kits

Buy these in bulk before you start. Running out mid-job means a trip to the chandlery
or a two-day Amazon wait.

| Kit | What's in it | Notes |
|---|---|---|
| Heat-shrink crimp terminal assortment (Red / Blue / Yellow) | Ring, spade, butt splice, and disconnect terminals in 22-10 AWG — adhesive-lined heat shrink | Amazon sells sorted trays by color. Buy extra butt connectors — you'll use more than you think |
| Bootlace ferrule assortment | Ferrules from 0.25 mm² to 6.0 mm², color-coded by size | Also sold as sorted trays. Your most-used sizes will be 0.5–1.0 mm² (22-18 AWG signal wire) |
| Adhesive-lined heat shrink tubing | Assorted diameters, 3:1 shrink ratio, marine-grade (adhesive lined) | Buy the adhesive-lined type, not plain. The adhesive seals out moisture at every joint |
| 18 AWG stranded wire — multiple colors | Signal wire for sensor runs. Get at least 4 colors beyond red and black (white, green, blue, yellow are common) | Red/black alone makes every signal run look identical. A third and fourth color let you distinguish signal pairs at a glance inside wire duct — worth it |
| Cable ties (assorted) | UV-rated for any exterior run, standard for interior | Bulk bag — you'll use hundreds |
| Wire labels (extra cartridges) | TZe tape for your label maker | At least two spare cartridges. You label more than you expect |

### Nice to have at Tier 3+

| Tool | What for | Budget |
|---|---|---|
| Hydraulic crimper (8 AWG to 4/0 AWG) | Crimping heavy cable lugs for battery cables, main power feeds, and distribution runs. The ratcheting crimper in the main list stops at about 10 AWG — this handles the big stuff up to 4/0 | $40-80 |
| USB RS-485 adapter (spare) | A second adapter for bus diagnostics from a laptop — your Pi's HAT is busy polling. Free tools like `modpoll` or the engine's `probe` command do the diagnosis | $10-25 |
| Cutoff wheel for your grinder | Cutting 35mm DIN rail to length — a cutoff wheel on the angle grinder you already have. No need for a specialty DIN rail shear | $5-10 |

---

## 10. Safety

### Before you start

- **Disconnect the battery bank** before working on any DC wiring. Batteries can deliver
  thousands of amps into a short circuit. A wrench across the terminals will weld itself
  to the posts and start a fire.
- **Fuse at the source.** Every new circuit gets a fuse or breaker where it connects to
  the power supply, not at the load end. The fuse protects the WIRE, not just the device.
- **Fire extinguisher within reach.** Not across the boat — within arm's reach of where
  you're working. Dry chemical (ABC rated) or CO2.

### While you work

- **One circuit at a time.** Don't energize a circuit until it's complete, inspected, and
  tested with a multimeter for continuity and shorts.
- **Ring out before you energize.** With the circuit de-energized, verify:
  - Continuity on every wire from end to end
  - No shorts between any conductor and ground
  - No shorts between conductors
  - Correct polarity on power connections
- **Photograph before you close.** Take photos of every terminal landing before closing
  an enclosure or wire duct. You will need them later.

### The reference build's fusing scheme

The reference build uses a tiered fusing approach:
- 60 A MEGA fuse at the battery (protects the main feed)
- 40 A breaker at the distribution panel (protects the automation branch)
- Individual 2-5 A fused terminal blocks (LED-indicator type) per module or sensor circuit

Each level protects the wire downstream of it. A sensor circuit fault blows a 2 A fuse,
not the 60 A main. The LED on the blown fuse lights up — you see which circuit failed
without pulling fuses.

---

## Applicable standards

- **ABYC E-11** — AC and DC Electrical Systems on Boats (the US standard)
- **ISO 10133** — Electrical installations on small craft (the international standard)
- **ABYC TE-4** — Wire and cable (marine-grade specification)

These standards are not free. If you're doing Tier 3+ wiring, buying a copy of ABYC E-11
is a worthwhile investment ($50-100 depending on format). Your AI session can help you
apply the standards but should not be your only source — verify critical decisions
(wire gauge for high-current runs, overcurrent protection sizing) against the standard.

---

## Reference

**Essential reading:**
- Victron "Wiring Unlimited" (free PDF from victronenergy.com) — DC fundamentals,
  connections, fusing, grounding, galvanic corrosion

**The worked example of a complete marine automation build:**
- `wiring-io/wiki/Wiring-Sheets.md` — the drawing system and all wiring sheets
- `wiring-io/wiki/Circuit-Register.md` — every circuit with build/proof status
- `wiring-io/Diagrams/` — YAML-to-HTML wiring sheet system (wire IDs, terminal landings)
- `wiring-io/wiki/Architecture.md` — the alarm/automation wiring architecture and principles
