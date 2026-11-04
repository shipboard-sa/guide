<!--
Purpose: Glossary and concept reference for non-EE/CS readers. Defines the terminology
  used throughout the guide and explains the key concepts needed to follow along.
Version: v1 (2026-11-04) — DRAFT.
Scope: PUBLIC — jargon-free definitions and short explainers.
Architecture: Reference page, not a workflow step. Linked from other pages where jargon
  first appears. External links point to free, stable resources for deeper reading.
Invocation: n/a (reference doc)
-->

# Glossary and Concepts

This page defines every technical term the guide uses and explains the concepts behind
them. You don't need to memorize this — use it as a lookup when something in the other
pages doesn't make sense.

The short definitions come first (alphabetical). The concept explainers follow — these
are the ideas worth reading before you start.

---

## Quick-reference glossary

| Term | What it is |
|---|---|
| **4-20 mA** | An analog signal where 4 milliamps = zero and 20 milliamps = full scale. Used by industrial sensors (tank level, pressure, temperature). The reason it starts at 4, not 0: a broken wire reads 0 mA, so the system can tell "sensor dead" from "sensor reads zero." |
| **AC** | Alternating current — what comes from shore power and generators. 120V/60Hz in the US, 230V/50Hz in most of the world. Your inverter converts DC to AC. |
| **ACK / acknowledge** | Pressing a button (physical or on-screen) to tell the system "I see the alarm, I'm on it." Some alarms clear on ack; others stay active until the condition clears. |
| **AI (analog input)** | A module channel that reads a continuously varying voltage or current (not just on/off). Used for tank levels, temperature probes, current sensors. See [Analog vs. digital](#analog-vs-digital). |
| **ABYC** | American Boat and Yacht Council — writes the US standards for marine electrical (E-11 is the big one). |
| **AIS** | Automatic Identification System — ships broadcast their position, speed, heading, and identity over VHF radio. Your AIS transponder transmits yours and receives others. **Class A** is required for commercial vessels (high power, faster updates, expensive — $1,500+). **Class B** is what recreational boats carry (lower power, slower updates, $300-800 — em-trak, Vesper, and similar brands). Almost all Class B transponders output NMEA 0183, not NMEA 2000. See the [BOM](04-Reference-BOM.md) for adapter guidance. |
| **Baud rate** | How fast devices talk on a serial bus, in bits per second. The reference build uses 9600 baud for Modbus. Every device on the bus must match. |
| **BMS** | Battery Management System — the brain inside a lithium battery bank that monitors each cell and protects the pack from over/under voltage, over-temperature, and over-current. |
| **Bonding system** | The network of green/yellow wires connecting all metal parts on the boat (through-hulls, engine, tanks, stanchions) to a common ground. Prevents galvanic corrosion between dissimilar metals and provides a fault return path for AC safety ground. |
| **Breaker** | A resettable overcurrent device. Flips off when too much current flows; you flip it back on. Unlike a fuse, it doesn't need replacing after it trips. |
| **Bus** | A shared communication wire (or pair of wires) that multiple devices connect to. RS-485, CAN, and NMEA 2000 are all buses. See [What is a bus?](#what-is-a-bus) |
| **CAN bus** | Controller Area Network — a two-wire differential bus designed for noisy environments. See [CAN bus on a boat](#can-bus-on-a-boat). |
| **Cerbo GX** | Victron's system monitor — reads battery state, solar, inverter/charger status, and publishes it all via MQTT. The automation system reads it as `cerbo.*` signals. |
| **Contactor** | A heavy-duty relay for high-current loads (hundreds of amps). Mechanically similar to a relay but bigger. The BatteryProtect is essentially a contactor with brains. |
| **Crimp** | A mechanical connection made by squeezing a metal terminal onto a wire with a special tool. The correct way to terminate wires on a boat (not solder, not wire nuts). |
| **Daisy chain** | Wiring topology where the cable goes from device A to device B to device C in a line. RS-485 requires daisy chain — not star (separate cables from a central point to each device). |
| **DC** | Direct current — what batteries provide. Your boat's house bank is 12V or 24V DC. |
| **Debounce** | Ignoring rapid on/off transitions from a sensor (caused by waves, vibration, or contact bounce). Implemented with the `for:` primitive in a rule — the condition must hold for N seconds before it counts. |
| **DI (digital input)** | A module channel that reads on or off — nothing in between. Connected to switches, float switches, reed contacts. See [Analog vs. digital](#analog-vs-digital). |
| **DIN rail** | A standard 35mm metal rail that clips into an enclosure. Modules, terminal blocks, fuses, and power supplies snap onto it. The universal mounting system for industrial automation. |
| **DO (digital output)** | A module channel that switches on or off — drives a relay coil or indicator. See [Analog vs. digital](#analog-vs-digital). |
| **Docker** | Software that runs applications in isolated containers on the Pi. Each service (rules engine, SignalK, Grafana) runs in its own container. If one crashes, the others keep running. |
| **Dry contact** | A switch or relay output with no voltage of its own — just two wires that are either connected (closed) or not (open). Most marine sensors are dry contacts. See [How dry contacts work](#how-dry-contacts-work). |
| **dry_run** | A rule that evaluates but doesn't act. It logs what it *would* do. Every new rule starts in dry_run until you've seen it behave correctly on real data. |
| **EMI** | Electromagnetic interference — electrical noise from motors, inverters, alternators, and VHF radios that can corrupt data on signal wires. Why you use shielded cable and keep signal wires away from power cables. |
| **Enclosure** | The box your DIN rail and modules live in. Rated by IP (ingress protection) — IP65 keeps out water spray, IP44 keeps out splashes. |
| **ETA** | In a rule message, `{eta:signal:target}` = estimated time for a value to reach a target at its current rate. "SoC at 15%, ~3.2 h to 50%." |
| **Ethernet gateway** | A device that converts RS-485 (Modbus) to Ethernet (TCP/IP). Lets the Pi talk to Modbus modules over the boat's network instead of a direct serial connection. |
| **Fail-safe** | The state a device returns to when power is removed. A normally-open relay drops out (opens) on power loss. Your system should be designed so that fail-safe = safe — losing the Pi doesn't leave the engine kill relay energized. |
| **Ferrule** | A small metal tube crimped onto the end of a stranded wire so it enters a screw terminal cleanly. Prevents strands from splaying. See page 05. |
| **Finding (F-nnn)** | A numbered record of something observed during commissioning. Could be a test result, an anomaly, a correction, or a discovery. |
| **Float switch** | A sensor that detects water level. A buoyant arm rises with the water and closes (or opens) a switch at a set height. Used for bilge pump monitoring and high-water alarms. |
| **Fuse** | A one-time overcurrent device — a thin conductor that melts when too much current flows, breaking the circuit. Protects the wire, not the device. Must be replaced after it blows. |
| **Galvanic corrosion** | Electrochemical attack that happens when two different metals are connected in saltwater. The less noble metal dissolves. Why zinc anodes exist, and why your bonding system matters. Read Victron "Wiring Unlimited" Chapter 8. |
| **Grafana** | Open-source dashboarding software. Connects to your history database and displays charts, gauges, and alerts. Runs on the Pi. [grafana.com](https://grafana.com/) |
| **Ground** | See [Three grounds on a boat](#three-grounds-on-a-boat) — it's not one thing. |
| **Heartbeat** | A periodic signal that says "I'm alive." The rules engine sends a heartbeat to the alarm panel. If the heartbeat stops, the panel lights a lamp. |
| **Hysteresis** | The gap between a threshold and its clear point. If an alarm fires at 20% SoC and clears at 25%, the 5% gap is the hysteresis band. Prevents the alarm from flapping on/off when the value hovers near the threshold. |
| **I/O** | Input/Output — the generic term for the module channels that read sensors (inputs) and drive relays (outputs). |
| **Inhibit** | When one rule blocks another. "During a flood, don't shed battery loads" — the flood rule inhibits the load-shedding rule because the pumps need power. |
| **Inverter** | Converts DC battery power to AC (120V or 230V) for household appliances. The MultiPlus is an inverter/charger — it does both directions. |
| **Isolated / non-isolated** | Whether two electrical circuits share a ground connection or are kept electrically separate. See [Isolated vs. non-isolated: why it matters](#isolated-vs-non-isolated-why-it-matters). |
| **io_master** | The YAML file that defines every I/O point in your system — every DI, DO, AI, and relay with its module address, function name, signal name, class, and state. The single source of truth for "what is connected where." |
| **IP rating** | Ingress Protection — a two-digit rating for enclosures. First digit = solids (dust), second = water. IP65 = dust-tight + water jets. IP44 = objects >1mm + splashing. |
| **Latch** | A condition that remembers: once set, it stays true until explicitly reset. "The engine has been running" (set when RPM goes up, reset when RPM stays down for 30 seconds). |
| **Live** | A circuit or rule that is in active service. A live rule acts on real conditions. A live circuit is read/driven by live rules. |
| **Modbus RTU** | A serial communication protocol that lets a master (the Pi or gateway) read from and write to slave devices (I/O modules) over RS-485. Simple, reliable, and decades old. See [What is Modbus?](#what-is-modbus). |
| **MQTT** | A lightweight messaging protocol. The Victron Cerbo publishes battery/solar/inverter data over MQTT. The automation system subscribes to the topics it needs. [mqtt.org](https://mqtt.org/) |
| **NMEA 0183** | The older marine data standard — plain-text sentences over a serial wire. Still found on many boats alongside (or instead of) NMEA 2000. See [NMEA 0183 vs. NMEA 2000](#nmea-0183-vs-nmea-2000). |
| **NMEA 2000 (N2K)** | The newer marine instrument network — binary messages over a CAN bus. Faster, multi-device, and self-describing. See [CAN bus on a boat](#can-bus-on-a-boat) and [NMEA 0183 vs. NMEA 2000](#nmea-0183-vs-nmea-2000). |
| **NC / normally closed** | See [Relay contacts: NO, NC, COM](#relay-contacts-no-nc-com). |
| **NO / normally open** | See [Relay contacts: NO, NC, COM](#relay-contacts-no-nc-com). |
| **OQ (Operational Qualification)** | The formal test that proves a circuit does what it's supposed to. Ring out → power up → verify function → record evidence. |
| **PGN** | Parameter Group Number — the message ID in NMEA 2000. PGN 127488 = engine RPM, PGN 127508 = battery status, PGN 129029 = GPS position. You don't need to memorize these; SignalK translates them to readable paths. |
| **Pi** | A Raspberry Pi — a small, inexpensive single-board computer that runs Linux. It hosts SignalK, the rules engine, Grafana, and the other software in your automation stack. |
| **PROTECT** | A relay class for safety-critical outputs (engine kill, charge disconnect). PROTECT relays require `clear: manual` on their rules, cannot be driven by simulated signals, and have the highest priority in relay arbitration. |
| **PTFIX** | A DIN-mount distribution block (Phoenix Contact brand name, now generic usage) that provides many connection points on a single terminal. Used for ground consolidation — one block gathers all the signal grounds and connects them to DC negative at a single point. |
| **Pull-to-ground** | See [Pull-to-ground: how digital inputs actually work](#pull-to-ground-how-digital-inputs-actually-work). |
| **Reed switch** | A sensor with two metal reeds inside a sealed glass tube. A magnet closes them. Used for door/hatch open detection (magnet on the door, switch on the frame) and dinghy theft alarms. |
| **Relay** | An electrically operated switch. See [What is a relay?](#what-is-a-relay) and [Relay contacts: NO, NC, COM](#relay-contacts-no-nc-com). |
| **Ring-out** | Testing every wire with a multimeter before applying power. Verifies continuity, isolation, and polarity. See page 07. |
| **RS-485** | A differential serial communication standard. Two wires (A and B) carry the signal; the voltage *difference* between them is the data, so noise that affects both wires equally cancels out. The physical layer under Modbus RTU. |
| **Rules engine** | The software that evaluates your rules every tick. It reads signals, checks conditions, and fires actions. It's generic — it doesn't know about boats. Your `rules.yml` teaches it what to watch and what to do. |
| **Shielded cable** | Cable with a metallic braid or foil wrap around the conductors. The shield blocks EMI. Ground it at one end only (the Pi/gateway end) to avoid ground loops. |
| **SeaTalk / SeaTalkng** | Raymarine's proprietary instrument networks. Original SeaTalk (1990s) is a single-wire protocol — not CAN, not NMEA 2000. SeaTalkng is NMEA 2000 compatible (same CAN protocol, different connector). See [Proprietary networks](#proprietary-networks-seatalkng-simnet-garmin). |
| **SignalK** | Open-source marine data server. Collects data from NMEA 2000, NMEA 0183, and other sources, and serves it as a unified data model over the network. The rules engine reads `sk.*` signals from it. [signalk.org](https://signalk.org/) |
| **SimNet** | Simrad/Navico's proprietary instrument network. NMEA 2000 compatible (same CAN protocol, different connector). Adapter cables to standard N2K connectors are readily available. See [Proprietary networks](#proprietary-networks-seatalkng-simnet-garmin). |
| **SoC** | State of Charge — how full your battery bank is, as a percentage. Reported by the BMS or the Victron Cerbo. |
| **Stale** | A signal whose last reading is too old. A sensor that stops reporting goes stale. The `stale` primitive detects this — "if no reading for 60 seconds, fire an alert." |
| **Star topology** | Wiring where separate cables run from a central point to each device. **Wrong** for RS-485 (causes signal reflections). Right for Ethernet. |
| **Termination resistor** | A 120-ohm resistor at each end of an RS-485 bus. Prevents signal reflections. The HAT or gateway has one (often a DIP switch); you add one at the last module. |
| **Thermistor** | A temperature sensor whose resistance changes with temperature. The Cerbo's temperature inputs use them. Not the same as a thermocouple (which generates a tiny voltage). |
| **Tick** | One evaluation cycle of the rules engine. Every tick, it reads all signals and checks all rules. Typically once per second. |
| **Twisted pair** | Two wires twisted around each other. The twist reduces electromagnetic interference by ensuring both wires pick up the same noise (which cancels out in a differential signal like RS-485). |
| **UNKNOWN** | The value of a signal that has never reported or whose last reading is too old. A condition on UNKNOWN is not met — the engine fails safe (doesn't fire) rather than false-alarming. |

---

## Concept explainers

### Analog vs. digital

The most fundamental distinction in automation:

**Digital** = on or off, 1 or 0, true or false. A float switch is either up or down.
A relay is either energized or not. There's no in-between.

- **DI (digital input):** reads a two-state sensor. "Is the bilge pump running?" → yes or no.
- **DO (digital output):** switches something on or off. "Turn on the engine room blower." → on or off.

**Analog** = a range of values. A tank level sensor reads anywhere from empty to full.
A temperature probe reads a number. A current sensor tells you how many amps are flowing.

- **AI (analog input):** reads a variable value. "What is the fuel tank level?" → 0-100%.

Most of your automation will be digital. Analog inputs come in at Tier 2+ for tank
levels, temperatures, and current sensing.

For a thorough intro: [SparkFun — Analog vs. Digital](https://learn.sparkfun.com/tutorials/analog-vs-digital)

#### The three types of analog signal

Analog sensors don't all speak the same language. There are three fundamentally
different ways a sensor reports a value, and they need different hardware to read:

**1. Resistive — the sensor changes its resistance**

The sensor is a variable resistor. Its resistance goes up or down with the thing
being measured. The A/D module sends a small known current through it and measures
the voltage to calculate resistance, then converts that to a reading.

| Sensor | How resistance changes | Typical range |
|---|---|---|
| Thermistor (temperature) | Resistance drops as temp rises (NTC) or rises (PTC) | 1 kΩ – 100 kΩ |
| Resistive tank sender | A float slides a wiper along a resistor strip | 33 – 240 Ω (US standard) or 10 – 180 Ω (European) |
| Strain gauge (load cell) | Resistance changes with mechanical strain | Tiny changes — needs an amplifier |

**The catch:** the A/D module must be designed for resistive input. It provides
an excitation current and measures the resulting voltage. A voltage-input module
can't read a resistive sensor directly — you'd need a signal conditioner in between.

**On your boat:** your engine's oil pressure and water temperature gauges are
probably resistive senders (they were designed for an analog gauge, not a computer).
The Victron Cerbo's temperature inputs expect 10 kΩ NTC thermistors.

**2. Voltage — the sensor outputs a voltage proportional to the value**

The sensor has its own electronics and outputs a voltage (typically 0–5V or 0–10V)
that maps linearly to the measured value. 0V = zero, 5V (or 10V) = full scale.

| Sensor | Typical output | Example |
|---|---|---|
| Pressure transducer | 0.5 – 4.5 V (ratiometric to 5V supply) | Oil pressure, fuel pressure |
| Hall-effect current sensor | 0 – 5 V proportional to amps | DC current monitoring |
| Some modern tank senders | 0 – 5 V | Aftermarket capacitive senders |

**The catch:** the A/D module must have a voltage input range that matches the
sensor's output. A module expecting 0–10V reads a 0–5V sensor at half scale (it
works, but you waste half your resolution). A module with a 0–5V input connected
to a 10V sensor saturates and may be damaged.

**Cable length matters more** with voltage signals than with current signals.
Voltage drop over a long cable run reduces the reading. Keep cable runs short, or
use thicker wire.

**3. Current loop (4–20 mA) — the sensor outputs a current**

The sensor adjusts the current flowing through a loop: 4 mA = zero, 20 mA = full
scale. The A/D module measures the current by reading the voltage across a precision
resistor (a "sense resistor" or "shunt").

| Property | Why 4–20 mA is used in industry |
|---|---|
| **Noise immunity** | Current doesn't change with cable resistance, so long runs don't affect accuracy |
| **Wire break detection** | A broken wire reads 0 mA, which is below the 4 mA "zero" — the system knows the sensor is dead, not reading zero |
| **Long cable runs** | Works reliably over hundreds of meters |
| **Two-wire** | Many 4–20 mA sensors are "loop-powered" — the same two wires carry both power and signal |

**The catch:** 4–20 mA sensors need a loop power supply (typically 24V). Some A/D
modules provide loop power on the AI channel; others expect you to supply it
externally. Check your module's spec before buying sensors.

**On your boat:** 4–20 mA is overkill for most runs under 10 meters but is the
right choice for an engine room tank sender running 15+ meters back to the nav
station, or for any sensor in a high-noise environment.

#### Which type does your module read?

**This is the critical question.** You can't plug a resistive sender into a
current-loop input — it won't read, and it could damage the sensor or the module.

| Module input type | Reads | Does NOT read |
|---|---|---|
| Resistive (excitation + sense) | Thermistors, tank senders, strain gauges | Voltage or current sensors |
| Voltage (0–5V, 0–10V) | Voltage-output sensors | Resistive or current sensors (without a shunt) |
| Current (4–20 mA) | Current-loop sensors | Resistive or voltage sensors |
| Configurable / universal | Multiple types via jumper or software | Depends on the config |

The Waveshare Analog Input module reads voltage (0–10V) and current (4–20 mA),
switchable per channel. It does not read resistive sensors directly. For
thermistors and resistive tank senders, you'd need either a signal conditioner
that converts resistance to voltage/current, or a module designed for resistive
input (like the Cerbo's dedicated temperature inputs).

**Your AI session should check** your sensor's output type against your module's
input type in Step 3 (Gap Analysis). A mismatch here is a wasted purchase.

---

### What is a relay?

A relay is a switch operated by electricity instead of your finger. Inside the box:

1. A **coil** of wire (an electromagnet)
2. A **spring-loaded arm** with a contact on the end
3. **Fixed contacts** that the arm touches

When you energize the coil (apply voltage), the electromagnet pulls the arm, moving
the contact from one position to another. When you de-energize the coil, the spring
pulls it back.

That's all a relay does: it uses a small electrical signal to switch a bigger one.
Your Modbus module's DO channel energizes the coil; the contacts switch the load
(a pump, a solenoid, an engine kill circuit).

**Why not just use a big switch?** Because the Pi can't flip a physical switch. The
relay is the bridge between software (a rule says "turn this on") and hardware (a
pump motor that draws 5 amps).

For a visual walkthrough: [YouTube — "How Relays Work" by The Engineering Mindset](https://www.youtube.com/watch?v=n4GiMnHMPaQ) (~10 min)

---

### Relay contacts: NO, NC, COM

Every relay has three terminals per pole:

```
        COM (Common)
         |
    ┌────┤
    │    ─┘  ← arm (moves)
    │
    ├── NO (Normally Open)     ← connected when coil is energized
    │
    └── NC (Normally Closed)   ← connected when coil is de-energized
```

| Terminal | State when coil is OFF (normal) | State when coil is ON (energized) |
|---|---|---|
| **COM → NC** | Connected (closed circuit) | Disconnected (open circuit) |
| **COM → NO** | Disconnected (open circuit) | Connected (closed circuit) |

**Why this matters for your boat:**

Think about what happens when the Pi dies and all relays lose power (de-energize):

- A **normally open** relay **opens** → the load turns OFF. Use NO when "off" is safe
  (a blower that should stop, a horn that should silence).
- A **normally closed** relay **closes** → the load turns ON (or the circuit completes).
  Use NC when you want the circuit to work BY DEFAULT and the relay only
  INTERRUPTS it. Example: a hold-off relay on a flow alarm — NC means the alarm
  path is complete unless the automation actively holds it off. Pi dies → hold-off
  drops → alarm works.

**The fail-safe question:** for each relay in your system, ask: "if the Pi dies right
now and this relay de-energizes, is the boat in a safe state?" The answer should
always be yes.

---

### How dry contacts work

A dry contact is a switch with no voltage of its own — just two wires that are
either touching (closed) or not (open). Most marine sensors are dry contacts:

- Float switches (bilge water level)
- Reed switches (door/hatch open)
- Pressure switches (oil pressure)
- Flow switches (raw water)
- Alarm relay outputs on other equipment (AIS, MultiPlus, Cerbo)

The Modbus module provides the voltage. The sensor just completes or breaks the circuit.

---

### Pull-to-ground: how digital inputs actually work

When you connect a float switch to a DI channel on a Modbus module, here's what's
happening electrically:

```
Module DI pin ──────── switch ──────── Module GND
      │
      └── internal pull-up resistor to V+
```

The module has a small resistor inside that "pulls" the DI pin up to its supply
voltage (say, 24V). This makes the pin read HIGH (logical 1) by default.

When the switch closes, it connects the DI pin to ground (0V). The pin gets
"pulled to ground" — it reads LOW (logical 0).

| Switch state | DI pin voltage | Module reads |
|---|---|---|
| Open (float down) | Pulled up to V+ by internal resistor | HIGH / 1 / TRUE |
| Closed (float up) | Pulled to ground through the switch | LOW / 0 / FALSE |

**Why "pull to ground" instead of "pull to high"?** It's more noise-immune. A
loose wire in a pull-to-ground system reads HIGH (open, no alarm) — a false
safe, not a false alarm. A loose wire in a pull-to-high system could float to any
voltage and trigger false alarms.

**The NC trick for safety sensors:** For a sensor where you want a broken wire to
trigger an alarm (not hide a real condition), use a normally closed contact:

| | NC contact | NO contact |
|---|---|---|
| Normal state | Closed → pin pulled to ground → reads LOW | Open → pin pulled high → reads HIGH |
| Alarm condition | Opens → pin goes HIGH | Closes → pin goes LOW |
| Broken wire | Opens → pin goes HIGH (same as alarm → **fail-safe**) | Opens → pin goes HIGH (same as normal → **fail-silent**) |

This is why the reference build's high-water sensors and fire/smoke detectors use
NC contacts — a broken wire looks like an alarm, not like "everything's fine."

---

### What is a bus?

A bus is a shared communication path. Instead of running a separate cable from
the Pi to every device, you run one cable (the bus) past all of them, and they
take turns talking.

```
Master ─────┬──────┬──────┬──── (termination)
            │      │      │
         Device 1  2      3
```

Rules of a bus:
- **One master, many slaves.** The master asks, a slave answers. Two masters talking
  at once = collision = garbled data.
- **Daisy chain, not star.** The cable goes from one device to the next in a line.
  Running separate cables from the master to each device creates signal reflections.
- **Termination.** A resistor at each end of the bus absorbs signals so they don't
  bounce back. On RS-485, it's 120 ohms.
- **Everyone agrees on speed.** Every device on the bus must use the same baud rate,
  data format, and addressing scheme.

The buses in your automation system: RS-485 (Modbus), CAN (NMEA 2000), and
optionally BMS-CAN (batteries). Each one follows these same principles with
different physical and protocol details.

---

### What is Modbus?

Modbus is a simple protocol from 1979 that is still the workhorse of industrial
automation. It defines how a master device (your Pi or gateway) reads from and
writes to slave devices (your I/O modules).

**Modbus RTU** runs over RS-485 (serial, two wires). The master sends a request:
"device 2, read your digital inputs." Device 2 answers with the values. This
happens once per second (or faster) for every module on the bus.

Each module has:
- An **address** (1-247) set by DIP switches or software
- **Registers** — numbered locations that hold values (coils = on/off, holding
  registers = numbers, input registers = read-only numbers)

When your rule reads `io.high_water_salon`, the engine translates that to "Modbus
address 1, read input register 2" (or whichever register your io_master maps it to).
You don't interact with Modbus directly — io_master handles the mapping.

For a visual intro: [YouTube — "What is Modbus?" by RealPars](https://www.youtube.com/watch?v=txi2p5_OjKU) (~8 min)

---

### NMEA 0183 vs. NMEA 2000

Most boats have one or both of these marine data standards. They carry the same
kinds of information (position, depth, wind, engine data) but they work completely
differently.

#### NMEA 0183 — the serial veteran (1983)

NMEA 0183 sends data as **human-readable text sentences** over a serial wire. If
you connect a terminal to a GPS with 0183 output, you see something like:

```
$GPGGA,123519,4807.038,N,01131.000,E,1,08,0.9,545.4,M,47.0,M,,*47
$GPRMC,123519,A,4807.038,N,01131.000,E,022.4,084.4,230394,003.1,W*6A
```

Each line is a "sentence" that starts with `$`, has a talker ID (`GP` = GPS),
a sentence type (`GGA` = fix data, `RMC` = recommended minimum), comma-separated
fields, and a checksum after `*`.

**Physical layer:**

| Property | NMEA 0183 |
|---|---|
| **Wires** | 2 (Tx+, Tx−) per talker — a differential pair, RS-422 levels |
| **Speed** | 4800 baud (standard) or 38400 baud (high-speed, for AIS) |
| **Topology** | Point-to-point: one talker, one or more listeners. NOT a shared bus |
| **Connectors** | No standard — bare wires, DB-9, various proprietary |

**The key limitation:** each 0183 output port is one talker to one (or a few)
listeners. If your GPS outputs 0183 and your chartplotter and your autopilot
both want it, you need a **multiplexer** (a device that combines multiple 0183
talkers onto one wire, or splits one talker to multiple listeners). On a boat
with many instruments, the wiring becomes a spaghetti of point-to-point serial
cables and multiplexers.

**What your automation reads:** SignalK understands NMEA 0183 natively. Connect
the 0183 output to a USB-serial adapter on the Pi, configure the baud rate in
SignalK, and the sentences appear as `sk.*` signals — the same namespace your
rules use. The AIS transponder on the reference build outputs alarm sentences
(`$AIALR`) over 0183 that SignalK translates to alarm paths.

#### NMEA 2000 — the networked successor (2000)

NMEA 2000 sends data as **binary messages** over a CAN bus (see [CAN bus on a
boat](#can-bus-on-a-boat)). Every device on the backbone can talk to every other
device — no multiplexers needed.

| Property | NMEA 2000 |
|---|---|
| **Wires** | 2 data (CAN-H, CAN-L) + shield + power (12V) + ground |
| **Speed** | 250 kbit/s (50× faster than 0183 standard) |
| **Topology** | Shared bus — all devices on one backbone with T-connectors |
| **Connectors** | Standardized Micro-C or Mini (DeviceNet) with colored rings |

**Advantages over 0183:** one cable for everything, faster updates (engine RPM
at 10 Hz vs. once per second), standardized connectors, device discovery (you
can see what's on the network), and bi-directional (devices can send commands,
not just data).

#### Side by side

| | NMEA 0183 | NMEA 2000 |
|---|---|---|
| **Data format** | ASCII text sentences | Binary PGNs |
| **Speed** | 4800 / 38400 baud | 250 kbit/s |
| **Wiring** | Point-to-point per talker | Shared backbone |
| **Connectors** | No standard | Standardized |
| **Devices on one wire** | 1 talker + listeners | 50+ |
| **Multiplexer needed** | Yes, for multiple talkers | No |
| **Your automation sees** | `sk.*` via SignalK | `sk.*` via SignalK |
| **Still on new boats** | Yes — AIS, some GPS, legacy instruments | Yes — most modern instruments |

#### Living with both

Many boats have a mix. Typical scenario:

- **N2K backbone** for the main instrument bus (chartplotter, depth, wind, engine
  gateway, autopilot)
- **0183 connections** for the AIS transponder (alarm sentences), an older GPS, or
  a legacy autopilot

SignalK unifies them. It reads from both 0183 and N2K sources and presents
everything through the same data model. Your rules don't know or care whether
`sk.navigation.position` came from a 0183 GPS or an N2K GPS — the signal is the
same.

**Bridges:** gateway devices exist that convert 0183 sentences to N2K PGNs and vice
versa, so an older 0183 instrument appears on the N2K network. However, the
reference build's experience with 0183-to-N2K gateway converters was poor —
reliability issues across multiple units. **The more reliable path for your
automation system** is USB-to-RS422 adapters feeding 0183 directly into SignalK.
The gateways are still useful if your *chartplotter* needs to see a 0183 device on
N2K (the chartplotter can't read SignalK), but for automation purposes, skip the
middleman.

[NMEA 0183 sentence reference — catb.org](http://catb.org/gpsd/NMEA.html) |
[Practical overview — Marinehowto](https://marinehowto.com/nmea-0183-beginners-guide/)

---

### CAN bus on a boat

CAN (Controller Area Network) is a two-wire bus designed for electrically noisy
environments — it was invented for cars in the 1980s. On your boat, CAN shows up
in at least two places, and possibly three:

#### NMEA 2000 (N2K)

Your instrument network. The depth sounder, wind sensor, GPS, AIS, autopilot, and
engine gateway all connect to a CAN bus with NMEA 2000 connectors (the round
micro-C or mini plugs with the colored rings). The data: position, speed, depth,
wind, engine RPM, fuel flow, battery voltage — everything your chartplotter shows.

- **Physical:** CAN bus at 250 kbit/s, with a backbone cable, T-connectors per
  device, and 120Ω terminators at each end
- **Power:** the backbone carries 12V to power small devices (GPS antennas, sensors)
- **Messages:** defined by PGN (Parameter Group Number) — each message type has a
  number
- **Your automation reads it** via SignalK, which translates PGNs into human-readable
  paths like `navigation.position` or `propulsion.main.revolutions`

If you have one CAN bus on your boat already, this is probably it.

[NMEA 2000 overview — Maretron](https://www.maretron.com/support/manuals/nmea-2000-overview/)

#### BMS-CAN

Your lithium battery's management system may have its own CAN bus for pack-to-pack
communication or for talking to the Victron Cerbo. This is physically the same kind
of cable (twisted pair, 120Ω termination) but carries battery-specific messages
(cell voltages, temperatures, charge/discharge limits).

- **Separate from N2K** — do not connect BMS-CAN to your NMEA 2000 backbone. They
  use different message definitions and will confuse each other.
- **Your automation reads it** via the SSM poller (which reads the BMS serial output)
  or via the Cerbo (which publishes battery data over MQTT). You don't read BMS-CAN
  directly.

#### Engine CAN (J1939)

Newer marine diesels (and most automotive engines) have an internal CAN bus using
the J1939 protocol. An engine gateway (like the Yacht Devices YDEG) translates J1939
messages to NMEA 2000 PGNs, so your instrument network and automation system can read
engine data. You connect the gateway to the engine's diagnostic port on one side and
the NMEA 2000 backbone on the other.

#### What they share

All three are CAN at the physical layer:

| Property | NMEA 2000 | BMS-CAN | J1939 (engine) |
|---|---|---|---|
| **Wires** | Twisted pair (CAN-H, CAN-L) | Twisted pair | Twisted pair |
| **Speed** | 250 kbit/s | Varies (250 or 500 kbit/s) | 250 kbit/s |
| **Termination** | 120Ω at each end | 120Ω at each end | 120Ω at each end |
| **Connectors** | Micro-C / Mini (DeviceNet) | Varies by manufacturer | Deutsch / OBD-II |
| **Messages** | PGNs (NMEA defined) | Manufacturer-specific | SAE J1939 |
| **Your automation sees** | SignalK (`sk.*` signals) | SSM poller or Cerbo (`ssm.*`, `cerbo.*`) | Via engine gateway → SignalK |

#### Proprietary networks (SeaTalkng, SimNet, Garmin)

The major marine electronics companies each have their own take on the instrument
network. The good news: they're all CAN underneath and mostly interoperate with
NMEA 2000. The bad news: the connectors and cables are different, so you need
adapters.

| Network | Manufacturer | Relationship to NMEA 2000 |
|---|---|---|
| **SeaTalk** (original) | Raymarine | Proprietary, NOT CAN. An older single-wire protocol from the 1990s. Does not connect to N2K without a converter (Raymarine SeaTalk to SeaTalkng converter, or a third-party gateway) |
| **SeaTalkng** | Raymarine | NMEA 2000 compatible at the protocol level — same PGNs, same CAN speed. Different connector (proprietary 5-pin vs. Micro-C). An adapter cable (SeaTalkng to DeviceNet) puts it on a standard N2K backbone |
| **SimNet** | Simrad / Navico (now Navico Group — also B&G, Lowrance) | NMEA 2000 compatible at the protocol level. Different connector (proprietary round yellow). Adapter cables to DeviceNet are available from Simrad and third parties |
| **Garmin NMEA 2000** | Garmin | Fully NMEA 2000 compliant. Uses standard Micro-C connectors — no adapter needed. Garmin sometimes adds proprietary PGNs for their own features, but standard data (GPS, depth, wind) is standard N2K |

**What this means for you:**

- If your boat has Raymarine instruments, you probably have a SeaTalkng backbone.
  You can connect standard N2K devices (including a gateway to the Pi) with a
  SeaTalkng-to-DeviceNet adapter cable (~$25).
- If your boat has Simrad/B&G, you probably have SimNet. Same story — adapter
  cable to DeviceNet.
- If your boat has Garmin, you're already on standard N2K connectors.
- If your boat has OLD Raymarine (SeaTalk, not SeaTalkng), you need a converter
  to get that data onto N2K or into SignalK. Alternatively, some SeaTalk devices
  have an NMEA 0183 output that SignalK can read directly.

**Your automation doesn't care** which brand's network carries the data — once
it reaches SignalK (via N2K or 0183), it's all `sk.*` signals. The brand
differences are a wiring and connector problem, not a software problem.

For a deeper dive: [YouTube — "CAN Bus Explained" by CSS Electronics](https://www.youtube.com/watch?v=FqLDpHsxvf8) (~10 min)

---

### Three grounds on a boat

"Ground" means different things depending on context. On a boat there are three,
and confusing them causes problems:

| Ground | What it does | Wire color | Where it connects |
|---|---|---|---|
| **AC safety ground** | Fault return path for shore power. If a hot wire touches a metal case, current flows through this wire and trips the breaker instead of through you | Green or green/yellow | Shore power → galvanic isolator → boat's bonding system (through-hulls, engine block, tanks) |
| **DC negative** | The return path for your 12V or 24V system. Every DC device's "minus" wire comes back here | Black (usually) | Battery negative terminal → negative bus → all DC loads |
| **Signal ground** | The voltage reference for your sensors and communication. Your Modbus module measures a DI as "voltage between the pin and GND" — GND is this | Per the module spec | Module GND terminal → PTFIX block → single point connection to DC negative |

**Why they're separate:** each ground carries different currents and different noise.
The engine starter pulls hundreds of amps through DC negative — you don't want that
current flowing through your RS-485 signal ground. Shore power can inject 60Hz noise
into the safety ground — you don't want that on your sensor circuits.

**The single-point connection:** your signal grounds collect at a PTFIX distribution
block and connect to DC negative at ONE point. This keeps the automation system's
ground clean. Noise from the shore power, inverter, or engine alternator stays on
the power ground, not on your signal lines.

Read Victron "Wiring Unlimited" Chapter 7.5 (Mobile Installations) and Chapter 8
(Galvanic Corrosion) for the full picture.

---

### Isolated vs. non-isolated: why it matters

When two circuits are **non-isolated**, they share a common ground. Current can
flow between them through that shared path. When they're **isolated**, there is
no electrical connection between them — they're separate worlds, linked only by
a signal that crosses the gap without a wire (using an optocoupler, a transformer,
or a relay contact).

```
NON-ISOLATED                        ISOLATED

  Module ──── Sensor                  Module ──── Sensor
    │                                   │      (optocoupler
    GND ─────── GND                     GND     or relay)     GND
    │                                                          │
    └── shared ground ──┘               (no connection)        │
                                                               └── separate ground
```

**Where you see this on a boat:**

**Modbus I/O modules** come in isolated and non-isolated variants. The Waveshare
relay modules, for example:

| Module | Isolation | What it means |
|---|---|---|
| Relay module (non-isolated inputs) | DI shares ground with the module | Simpler wiring, but noise on the sensor circuit can reach the bus |
| Relay module (isolated inputs) | DI has its own ground, optically separated from the bus | Sensor noise can't reach the bus; each input can reference a different voltage |

**RS-485 ports** on gateways and HATs may be isolated or not. An isolated RS-485
port means the bus ground is separated from the device's logic ground — noise and
ground potential differences on the bus can't upset the gateway's processor.

**When isolation matters most:**

1. **Long cable runs.** The ground potential at the engine room end of a 15-meter
   cable may differ from the nav station end by a volt or more (voltage drop from
   other equipment sharing the return path). Non-isolated inputs see this as a DC
   offset on their signal. Isolated inputs don't care — they measure the voltage
   between their own two terminals.

2. **Mixed voltage systems.** Your boat has 12V and 24V circuits (and maybe 48V
   from the batteries). An isolated input can read a 24V sensor while the module
   runs on 12V. A non-isolated input needs the sensor on the same voltage system
   as the module.

3. **Noise rejection.** The engine room is electrically noisy (starter motor,
   alternator, fuel pump). Isolation prevents noise on one sensor's wiring from
   propagating through the shared ground to every other sensor on the module.

4. **Ground loops.** When two devices are connected by both a signal wire AND a
   separate ground path (through the bonding system, through the DIN rail, through
   the N2K backbone), current can circulate in the loop and inject noise. Isolation
   breaks the loop.

**The practical rule:** for a small system (Tier 2, all modules in one enclosure,
short cable runs), non-isolated modules are fine and cheaper. For a larger system
(Tier 3+, modules in multiple locations, long runs to the engine room), isolated
inputs on your sensor modules are worth the extra cost.

For a visual explanation: [YouTube — "Isolation in Industrial I/O" by Acromag](https://www.youtube.com/watch?v=FN4c7FdMTZs) (~5 min)

---

### Signal-to-noise and why shielding matters

Your automation wires carry tiny signals (milliamps, millivolts). Your inverter,
engine alternator, VHF radio, and shore power cable carry big, noisy signals.
When a big noisy cable runs near a small signal cable, the noise couples into
the signal — this is EMI (electromagnetic interference).

On an RS-485 bus, noise shows up as garbled data (CRC errors, timeouts). On an
analog sensor, it shows up as jittery readings. On a digital input, it can cause
false triggers.

**Defenses:**

1. **Separation.** Keep signal cables 12+ inches from power cables. Where they
   must cross, cross at 90 degrees.
2. **Shielded cable.** A metallic braid or foil around the conductors blocks
   external noise. Ground the shield at ONE end (the Pi/gateway end) — grounding
   both ends creates a ground loop that makes things worse.
3. **Twisted pair.** The twist ensures both wires pick up the same noise, and
   the differential receiver (RS-485) cancels it out.
4. **Filtering (debounce).** The `for:` primitive in your rules ignores transient
   false readings — the condition must hold for N seconds before it counts.

---

## Further reading

| Topic | Resource | Why |
|---|---|---|
| DC fundamentals, fusing, grounding | [Victron "Wiring Unlimited"](https://www.victronenergy.com/upload/documents/Wiring-Unlimited-EN.pdf) (free PDF, 69 pages) | The single best free resource for marine DC wiring |
| Marine electrical standards | ABYC E-11 ($50-100) | The US reference standard; worth buying for Tier 3+ |
| Modbus protocol | [Simply Modbus](https://www.simplymodbus.ca/) | Clear, free introduction with examples |
| NMEA 0183 | [Marinehowto NMEA 0183 guide](https://marinehowto.com/nmea-0183-beginners-guide/) | Practical beginner's guide with wiring diagrams |
| NMEA 2000 | [Maretron NMEA 2000 overview](https://www.maretron.com/support/manuals/nmea-2000-overview/) | Practical guide from a marine electronics company |
| CAN bus | [CSS Electronics CAN bus intro](https://www.csselectronics.com/pages/can-bus-simple-intro-tutorial) | Visual, well-paced for non-engineers |
| SignalK | [signalk.org](https://signalk.org/) | The project site, with documentation and server downloads |
| Raspberry Pi | [raspberrypi.com/documentation](https://www.raspberrypi.com/documentation/) | Official docs — start with "Getting Started" |
| Relay fundamentals | [The Engineering Mindset — How Relays Work](https://www.youtube.com/watch?v=n4GiMnHMPaQ) | 10-minute video, clear animations |
| RS-485 wiring | [Lammert Bies — RS-485](https://www.lammertbies.nl/comm/info/RS-485) | Short, practical, covers termination and topology |
