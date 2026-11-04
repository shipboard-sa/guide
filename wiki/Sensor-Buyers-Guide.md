<!--
Purpose: Buyer's guide to sensors for marine automation — every category an automation
  system might monitor, with interface types, products, and integration notes.
Version: v1 (2026-11-04)
Scope: PUBLIC — sensor selection guide for new owners.
Architecture: Sensors feed into the Modbus I/O layer (DI, AI) or the N2K bus. Each
  category explains which interface applies.
Invocation: n/a (reference doc)
-->

# Sensor Buyer's Guide

Your automation system is only as good as its sensors. This page covers every sensor
category a boat automation system might need, organized by what you're monitoring. Each
section lists the interface type (dry contact → Modbus DI, analog → Modbus AI, N2K, or
Bluetooth) and specific products to consider.

**Aqualarm** (aqualarm.net) makes the widest range of marine-specific sensors — bilge,
fire, cooling water, oil pressure, and engine temperature in one product line. They're the
default starting point for any category they cover. The gap is engine sensors: oil pressure
and coolant temperature senders for specific engines are OEM or aftermarket parts, not
Aqualarm's territory.

---

## Interface types at a glance

Before shopping, understand how each sensor talks to your system:

| Interface | Connects to | Signal type | Example |
|---|---|---|---|
| **Dry contact** (NC or NO) | Modbus DI (digital input) | Switch opens or closes | Float switch, reed switch, pressure switch, thermal snap switch |
| **4-20mA / 0-5V** | Modbus AI (analog input) | Proportional current or voltage | Pressure transmitter, Hall effect CT, variable tank sender |
| **Resistive** (33-240Ω or 10-180Ω) | N2K converter or gauge | Variable resistance | Most OEM tank senders, engine gauge senders |
| **NMEA 2000** | N2K bus → SignalK | Digital PGN messages | Maretron, Navico, Digital Yacht products |
| **Bluetooth** | Cerbo GX or Pi BLE | Wireless | RuuviTag (temperature), Victron SmartShunt |
| **Thermocouple** | Dedicated monitor or Modbus RTU thermocouple module | mV signal | Exhaust gas temperature (EGT) probes |

**Rule of thumb:** For simple on/off alarms (bilge high, fire, door open), use dry-contact
sensors into Modbus DIs. For proportional measurements (how much current, what temperature,
what level), use 4-20mA sensors into Modbus AIs. For data already on your N2K bus (wind,
depth, SOG), read it via SignalK — don't duplicate it.

---

## What each sensor type is worth to your system

Before you shop by category, understand what each sensor *type* can do for you. The
same $15-80 sensor gives you very different capabilities depending on whether it's a
simple switch or a proportional measurement. This section is the education — it's how
you start thinking about what's possible, not just what's needed.

### What a generic Tier 3 build looks like

This is the bird's-eye view — what systems get what sensors and what you gain from each.
The details for every sensor type follow in the sections below.

| System | Sensors | What you gain |
|---|---|---|
| **Bilge** | Float switch (DI), CT clamp on pump (AI) | High water alarm + pump runtime, duty cycle, running dry or running hard |
| **Fire / smoke** | Smoke detector (DI), thermal snap disc (DI), liquid detection (DI) | Engine room fire alarm, high-temp alarm, fuel/coolant leak detection |
| **Engine room** | Temperature probes (AI), pressure transmitter (AI), CT clamps (AI), liquid detection (DI) | Exhaust gas trend, oil/coolant pressure trend, alternator output, coolant/fuel leak |
| **Pumps** (raw water, fresh water, washdown, etc.) | CT clamp (AI), flow switch or float (DI) | Running/stopped, running hard, running dry, duty cycle, current draw trend |
| **Tanks** (fuel, water, waste) | Resistive sender → Cerbo or N2K (existing), optional CT on transfer pump | Level monitoring, fill alerts with ETA, imbalance warning, pump health |
| **Power** | Victron ecosystem (Cerbo, SmartShunt, MPPTs, MultiPlus) via N2K/Venus | Bank SoC, charge/discharge, solar yield, shore power, load — already on the bus |
| **Refrigeration / freezer** | Temperature probe (AI), CT clamp on compressor (AI) | Box temp trend, compressor duty cycle, compressor drawing too much or too little |
| **Cabin comfort** | Temperature + humidity (AI or I²C) | Condensation risk, ventilation decisions, heating/cooling confirmation |
| **Navigation** | Wind, depth, speed, GPS — typically already on N2K | Course, weather, depth alarm, anchor watch — read via SignalK, don't duplicate |
| **Security** | Reed switches on hatches/doors (DI) | Open/closed state, "left open" timer alerts, departure checklist verification |
| **Rigging / structural** | Accelerometer, load cell (Tier 4+) | Anchor rode tension, mast/rigging loads, motion severity — emerging, not mainstream yet |

Most of this is $15-80 per sensor plus a Waveshare I/O module channel. The Victron
power stack is the expensive piece, but if you're on a Victron boat it's already there.

### Dry-contact sensors (switches) — one bit of information

A switch is open or closed. That's one bit. But one bit, combined with the rules
engine, gives you more than you'd expect:

| What the sensor tells you | What the rules engine adds | Example |
|---|---|---|
| Something is ON or OFF right now | **Duration** — how long has it been on? | Bilge pump running > 5 minutes = water ingress |
| A threshold was crossed | **Debounce** — was it real or a transient? | Float switch active > 3 seconds = genuine high water, not wave slosh |
| A condition exists | **Cycling detection** — how often does it toggle? | Pump starts 6 times in 60 seconds = stuck float or losing prime |
| A safety limit was reached | **Fail-safe wiring** — NC contact means a broken wire also alarms | Oil pressure switch opens = low oil OR broken wire, both demand attention |
| Something changed state | **State correlation** — combine with other signals | Door opens + anchor alarm armed + nighttime = security alert |

**Common dry-contact sensors on a boat:**

| Sensor | What it detects | Typical use |
|---|---|---|
| **Float switch** | Water level — floats up, contact closes | Bilge high water alarm, tank overflow, sump level |
| **Flow switch** | Liquid flow — vane deflects, contact closes | Raw water cooling flow (the fastest engine protection sensor you can buy) |
| **Pressure switch** | Pressure above/below a setpoint | Engine oil pressure alarm, fresh water pump cycling, hydraulic system |
| **Thermal snap switch** | Temperature above a setpoint — bimetallic disc snaps open | Exhaust overheat, engine block overheat, bearing overheat |
| **Reed switch + magnet** | Proximity — magnet moves away, contact opens | Door/hatch open, seacock position, locker security, window open |
| **Fire/heat detector** | Fixed temperature or rate-of-rise | Engine room fire, galley fire, electrical panel fire |
| **Smoke detector (with relay)** | Smoke particles | Accommodation fire — must have relay output for automation integration |
| **Limit switch** | Mechanical position — arm deflects at end of travel | Windlass chain counter (counts per link), hatch fully open/closed |
| **Tilt switch** | Angle change — mercury or ball rolls to contact | Bilge pump (some types), equipment tamper detection |
| **Vibration switch** | Mechanical vibration above a threshold | Pump running/not running (crude but simple), engine running confirmation |

**Cost:** $3-120 per sensor. The cheapest and most reliable sensor type — no electronics,
no power supply, no calibration.

### Analog sensors (4-20mA, 0-5V, 0-10V) — proportional measurement

An analog sensor gives you a continuous value, not just on/off. That value unlocks
an entirely different class of monitoring:

| What the sensor tells you | What the rules engine adds | Example |
|---|---|---|
| The current value of a measurement | **Threshold alarms** — alert at a specific level | Battery bank below 20% SOC |
| How much, not just whether | **Hysteresis** — different set and clear points prevent flapping | Alert at 20%, clear at 25% — no alarm chatter |
| A proportional reading | **Rate of change** — how fast is it changing? | Barometric pressure falling 3 hPa/hour = weather incoming |
| A continuous signal | **Trend analysis** — what's the trajectory? | Fuel consumption trending up = hull fouling or prop issue |
| A measurement range | **ETA calculation** — when will it reach a threshold? | Battery at 35% and falling — estimated 4 hours to critical |
| Current draw of a load | **Operational profiling** — what's normal vs abnormal? | Bilge pump drawing 2A (normal) vs 4A (running hard against head) vs 0.5A (running dry) |

**Common analog sensors on a boat:**

| Sensor | What it measures | Typical use |
|---|---|---|
| **Hall effect CT clamp** | Current draw (amps) | Pump runtime and health, compressor duty cycle, alternator output (see dedicated section below) |
| **Pressure transmitter** | Continuous pressure (PSI/bar) | Engine oil pressure trend, raw water pressure, fuel pressure, hydraulic pressure |
| **Temperature probe (RTD/thermistor)** | Continuous temperature | Exhaust gas temp trend, engine coolant, battery compartment, refrigeration |
| **Tank level sender (resistive)** | Fluid level (% full) | Fuel, water, waste, oil — most OEM senders are resistive; read via N2K converter or voltage divider → Modbus AI for faster polling |
| **Ultrasonic level sensor** | Distance to fluid surface | Non-contact tank level — mounts on top, nothing inside the tank |
| **Humidity sensor** | Relative humidity (%) | Condensation risk in lockers, engine room, battery compartment |
| **Flow meter (turbine/paddle)** | Flow rate (GPM/LPM) | Fresh water consumption tracking, raw water flow rate (proportional, not just on/off) |
| **Load cell / strain gauge** | Force/weight | Anchor rode tension, rigging load, tank weight (alternative to level sender) |
| **Voltage sensor** | Voltage (continuous) | Battery bank voltage trending, solar panel output, alternator field voltage |
| **pH sensor** | Acidity/alkalinity | Watermaker output quality, holding tank monitoring |

**Cost:** $8-300 per sensor. Requires an analog input module (Modbus AI). Worth the
extra cost when you need to know *how much*, not just *whether*.

### Current sensors (Hall effect CT clamps) — the most versatile single sensor

A current sensor deserves its own callout because it's the single most capable sensor
type for the money. A $15 split-core CT clamp on a pump wire — no circuit modification,
just clamp it around the wire — gives you:

| Detection | How | Value |
|---|---|---|
| **Running / not running** | Current > 0 | Basic monitoring — same as a dry contact, but no wiring into the pump circuit |
| **Runtime duration** | How long current > 0 | Water ingress rate, compressor duty cycle |
| **Cycling frequency** | How often it toggles | Stuck float switch, pump losing prime, short-cycling compressor |
| **Running hard** | Current higher than normal | High head pressure (water level above pump), restricted discharge |
| **Running dry** | Current lower than normal | No water to pump, air lock, suction blockage |
| **Stalled / seized** | Current spike then zero | Motor failure, locked rotor |
| **Not running when it should be** | Zero current when float switch is active | Tripped breaker, dead motor, failed connection |

One sensor, seven detection modes, $15, no circuit modification. Put one on every
pump and compressor.

### Temperature sensors — context changes everything

Temperature is the most broadly useful measurement on a boat. The same sensor type
tells you very different things depending on where you put it:

| Location | What it tells you | Why it matters |
|---|---|---|
| **Fridge / freezer** | Compressor health, door left open, thermostat failure | You find out before you lose $500 in provisions |
| **Engine room ambient** | Overheat, fire precursor, ventilation failure | Early warning before a fire detector trips — minutes of lead time |
| **Engine exhaust** | Cooling system health, impeller wear, blockage | Trend = slow degradation that a flow switch won't catch |
| **Engine coolant** | Overheat, thermostat stuck, coolant loss | The classic engine alarm — but with trend data you catch it earlier |
| **Battery compartment** | Thermal runaway precursor (lithium), overcharging (lead-acid) | The temperature alarm that prevents a fire |
| **Cabin / berth** | Comfort, HVAC performance, ventilation | Crew comfort and condensation management |
| **Lazarette / storage** | Heat buildup, ventilation failure | Protects provisioning and electronics stored aft |
| **Waterline / through-hull** | Sea water temperature for navigation, fishing, weather | Already on N2K for most boats — just read it |
| **Bilge** | Water temperature changes indicating inflow | Cold water in the bilge when it should be ambient = hull leak or fitting failure |
| **Rigging / mast base** | No practical sensor here — but standing rigging tension correlates with temperature | Future: synthetic rigging load monitoring |

### Pressure sensors — know the force

| Application | Sensor type | What it tells you |
|---|---|---|
| **Engine oil pressure** | Pressure switch (alarm) or transmitter (continuous) | Switch: binary alarm. Transmitter: pressure trend at different RPMs — early bearing wear shows as dropping pressure under load |
| **Fuel system pressure** | Pressure transmitter | Fuel filter clogging (pressure drop), injection system health |
| **Raw water system** | Pressure switch or transmitter | Strainer clog (pressure drop before pump), pump output (pressure after pump) |
| **Hydraulic system** (autopilot, windlass) | Pressure transmitter | System health, leak detection (pressure decay), load profiling |
| **Barometric** | N2K environmental sensor | Weather forecasting — rate of change is the key signal |
| **Fresh water system** | Pressure switch | Accumulator health, pump cycling (pressure switch chatter = waterlogged accumulator) |

### Vibration and accelerometer sensors — feel the machine

An emerging category that's becoming affordable:

| Application | What it tells you | Why it matters |
|---|---|---|
| **Engine mounts** | Vibration signature changes over time | Worn mounts, misalignment, cylinder imbalance — detectable before audible |
| **Shaft / coupling** | Rotational vibration, bearing wear | Cutlass bearing wear, prop damage, shaft alignment |
| **Pump motors** | Bearing wear, cavitation | Failing bearings have a distinctive vibration signature weeks before failure |
| **Hull (wave impact)** | Slam loading in heavy weather | Structural monitoring — how hard is the boat working? |

These are Tier 4+ territory — the sensors exist ($20-50 for MEMS accelerometers) but
the analysis requires more sophisticated signal processing than simple threshold rules.
Worth knowing about as the technology matures.

### Environmental sensors — the boat's weather station

| Measurement | Where it matters | What it gives you |
|---|---|---|
| **Humidity** | Engine room, lockers, bilge spaces | Condensation risk, mold prevention, ventilation scheduling |
| **Dew point** | Calculated from temp + humidity | The actual condensation threshold — more useful than humidity alone |
| **Light level** | Cockpit, cabin | Day/night automation (anchor light, courtesy lights), solar performance |
| **Air quality / VOC** | Engine room, battery compartment | Fuel vapor detection, battery off-gassing (lead-acid hydrogen) |
| **UV index** | Cockpit / deck | Canvas and gelcoat degradation tracking |

### Liquid detection sensors — beyond the bilge

| Application | Sensor type | What it tells you |
|---|---|---|
| **Fuel leak** | Hydrocarbon-sensing probe or absorbent rope sensor | Diesel in the bilge or engine drip pan — environmental and fire risk |
| **Coolant leak** | Conductivity probe in drip tray | Antifreeze leak at the engine — catch it in the drip tray before it reaches the bilge |
| **Refrigerant leak** | Refrigerant-specific gas sensor | AC or refrigeration charge loss — expensive refrigerant and declining cooling |
| **Black/grey tank overflow** | Float switch in overflow path | Overfull holding tank before it backs up into the boat |

### Position and motion sensors

| Sensor | Application | Interface |
|---|---|---|
| **GPS** | Anchor drag detection, geofencing, position logging | Already on N2K or via SignalK |
| **IMU (gyro + accelerometer)** | Roll/pitch monitoring, heel angle | I2C/SPI → Pi, or N2K (Maretron) |
| **Compass / heading** | Autopilot input, heading logging | N2K (fluxgate or GPS compass) |
| **Wind (apparent + true)** | Sail trim, weather monitoring, anchor alarm enhancement | N2K masthead unit → SignalK |
| **Depth / sounder** | Anchor alarm grounding protection, bottom profiling | N2K transducer → SignalK |
| **Speed (through water)** | Leeway calculation, hull fouling detection (SOG vs STW divergence) | N2K paddlewheel or ultrasonic → SignalK |
| **Rudder position** | Autopilot feedback, steering system health | N2K rudder reference → SignalK |

Most of these are already on your boat from the navigation instruments. The automation
value is reading them into the rules engine — an anchor alarm that uses GPS + wind +
depth + rode scope is far more capable than GPS alone.

### Camera and image sensors (Tier 3/4+)

Most boats won't have this capability yet — it requires network bandwidth, storage,
and processing power that a basic automation build doesn't include.

| Application | What it gives you |
|---|---|
| **Cockpit / deck** | Person detection (security), weather monitoring, departure/arrival recording |
| **Engine room** | Visual inspection without opening the hatch — check for leaks, smoke, displaced equipment |
| **Mast camera** | Rigging inspection, AIS vessel identification, weather horizon |
| **Underwater (hull)** | Hull fouling assessment, through-hull inspection, prop check |

Cameras are data-heavy (network bandwidth, storage) and the AI person-detection
capability is still maturing. Worth knowing about as the ecosystem develops.

---

## 1. Bilge and water detection

The most critical sensor category. A bilge alarm is the minimum viable automation.

### Dry-contact float switches (→ Modbus DI)

| Product | Type | Interface | ~Price | Notes |
|---|---|---|---|---|
| **Aqualarm 208A** high water detector | Electronic (no moving parts) | Dry contact (NO) | $50-80 | Patented field-effect sensing, no float to stick. Mount above your automatic float switch level |
| **Rule 33ALA** bilge alarm float switch | Mechanical float | Dry contact (NO) | $20-30 | Simple mercury-free float. Works but mechanical floats can stick in dirty bilges |
| **Johnson Pump Bilge Alert** | Electronic (no moving parts) | Dry contact alarm output + built-in buzzer | $40-60 | Self-contained alarm with a sensor; wire the dry contact to a Modbus DI for automation |
| **Seaflo high water alarm** | Mechanical float + alarm | Dry contact + built-in buzzer (85 dB) | $15-25 | Budget option; adequate for a secondary bilge |

**Wiring pattern:** Float switch → Modbus DI. Rule: "if DI is active for > 3 seconds,
alert." The time filter avoids false triggers from wave slosh.

**Best practice:** Mount the alarm sensor ABOVE the bilge pump's automatic float switch.
The pump handles normal water; the alarm fires only when the pump can't keep up.

### Pump runtime monitoring (→ Modbus AI)

Instead of (or in addition to) a water-level sensor, monitor the bilge pump's power draw
with a Hall effect current sensor. This tells you the pump is running, how long it runs,
and how often it cycles — all of which indicate water ingress rate.

See the [Current sensing](#5-current-sensing) section below.

---

## 2. Fire, smoke, and CO detection

### Fire / heat detectors (→ Modbus DI)

| Product | Type | Interface | ~Price | Notes |
|---|---|---|---|---|
| **Aqualarm fire alarm** (various models) | Fixed-temperature heat detector | Dry contact | $30-60 | Marine-rated, 12V. Part of the Aqualarm 5-detector engine monitoring kit |
| **Fireboy-Xintex FBD-2-R** two-zone panel | Heat + smoke detection panel | Relay output (2A) + transistor fault output | $200-350 | Monitors up to 14 detectors (12V) or 8 (24V). Relay output wires to a Modbus DI for automation integration |
| **Maretron SH-003** smoke/heat detector kit | Combined smoke + heat + relay | Dry contact via auxiliary relay | $150-250 | Includes relay and sounder. The relay output is your integration point — wire to a Modbus DI |

**Important:** Marine fire detectors must be designed for the marine environment (vibration,
humidity, salt). A residential smoke detector may not survive and may not meet ABYC
standards. Use marine-rated products.

### CO detectors

Most marine CO detectors are self-contained alarm units (Fireboy-Xintex, Safe-T-Alert).
Some have a relay or dry-contact output for integration; many do not. If yours has no
output, your automation can still detect CO indirectly — monitor the detector's power draw
or add a dedicated CO sensor with a relay output.

---

## 3. Engine sensors — the gap

This is the category Aqualarm doesn't fully cover. Engine monitoring requires sensors
matched to YOUR engine's sender threads, pressure ranges, and temperature ranges. These
are typically OEM or aftermarket engine parts, not generic marine sensors.

### What to monitor on a diesel engine

| Parameter | Sensor type | Interface | Why |
|---|---|---|---|
| **Raw water flow** | Flow switch (in-line) | Dry contact → Modbus DI | Earliest warning of impeller failure, strainer clog, or hose collapse |
| **Exhaust temperature** | Thermocouple or thermal switch | Thermocouple → monitor, or snap switch → DI | Second warning of cooling failure (fires after flow stops) |
| **Oil pressure** | Pressure switch (alarm) or pressure transmitter (gauge) | Switch → DI, or 4-20mA → AI | Low oil = engine damage. OEM alarm switch is the minimum |
| **Coolant temperature** | Temp switch (alarm) or temp sender (gauge) | Switch → DI, or resistive → gauge/N2K | Overheat warning. Usually OEM — don't replace what works |
| **Alternator output** | Voltage sense at the house bank | Via Victron/N2K or analog → AI | Confirms the alternator is charging. SignalK usually has this already |

### Raw water flow switches

The most important engine sensor you can add. It alarms BEFORE the engine overheats.

| Product | Thread | Flow range | Interface | ~Price | Notes |
|---|---|---|---|---|---|
| **Aqualarm 13209** | 1" barbed (hose) | 1.5-12 GPM (adjustable) | Dry contact (NC, fail-safe) | $80-120 | NC means key-on tests the circuit automatically. Upgraded 3A switch |
| **Aqualarm 10233** | 1" female NPT | 1.5-12 GPM | Dry contact (NC) | $80-120 | Pipe-thread version for hard plumbing |
| **Aqualarm 10271** | 1.5" female NPT | Higher flow | Dry contact (NC) | $90-130 | For larger engines |
| **Aqualarm 10331** | 2" barbed | Higher flow | Dry contact (NC) | $90-130 | For high-flow raw water circuits |

**Wiring:** NC contact → Modbus DI. When flow stops, the contact opens → DI state changes
→ rule fires → engine kill relay and phone alert. NC wiring is fail-safe: a broken wire
also triggers the alarm.

**Mount location:** As close to the raw water pump as practical, above the waterline.
Periodic cleaning recommended — sea water crud can foul the vane.

### Exhaust temperature

| Product | Type | Interface | ~Price | Notes |
|---|---|---|---|---|
| **NASA (Clipper) EX-1** exhaust monitor | K-type thermocouple + display + alarm | Alarm relay output | $100-150 | Self-contained monitor with configurable alarm threshold. Wire the relay to a Modbus DI. Set threshold ~10°C above normal operating exhaust temp |
| **Oceanic Systems 4510** EGT monitor | Thermocouple + NMEA display | Alarm output | $120-180 | Includes probe; reads on NMEA displays |
| **SMX thermal alarm switch** | Snap switch, 180°F | Dry contact (NC) | $25-40 | Simple: bolts to the exhaust elbow. No electronics, no display, just a switch. Wire to Modbus DI |

**Why both flow AND exhaust?** A flow switch alarms faster at high loads (immediate when
flow stops). An exhaust sensor catches slow degradation (partial blockage, gradual impeller
wear). Together, they cover the failure modes.

### OEM engine alarm switches (oil pressure, coolant temp)

These are engine-specific. For a Yanmar 4JH-series:

| Part | OEM number | What it does | Interface | ~Price |
|---|---|---|---|---|
| Oil pressure switch | 119773-91501 | Opens below ~5 PSI | Dry contact | $30-60 |
| Coolant temperature switch | 127610-91350 (or 129927-44900 for 4JH80) | Opens above ~105°C | Dry contact | $30-50 |
| Oil pressure sender (for gauge) | 129A00-57100 (4JH80) | Variable resistance | Resistive → gauge or N2K converter | $40-80 |

**If your engine already has alarm switches:** Don't replace them. Tap them — wire the
existing switch in parallel to a Modbus DI (or use the alarm panel's parallel input
design from Tier 3). The OEM alarm stays functional and your automation reads the same
signal.

**If your engine has NO alarm switches:** Check the engine manual for available ports. Most
diesel engines have tapped bosses for oil pressure, coolant temperature, and exhaust
temperature sensors. Yanmar, Volvo Penta, and Perkins all publish which ports are
available.

### Engine gateways (J1939 / NMEA 2000)

Modern engines with electronic injection (common rail) often have a J1939 CAN bus that
carries oil pressure, coolant temp, RPM, fuel rate, and more as digital data. A gateway
converts this to NMEA 2000 for your SignalK system.

| Product | Input | Output | ~Price | Notes |
|---|---|---|---|---|
| **Veratron EngineBox** | Analog engine senders + J1939 | NMEA 2000 | $250-400 | Converts both analog senders and J1939 to N2K. Good for mixed old/new engines |
| **Yacht Devices YDEG-04** | J1939 | NMEA 2000 | $200-300 | J1939-to-N2K gateway. Only works if your engine has a J1939 bus (check your manual) |
| **Maretron J2K100** | J1939 | NMEA 2000 | $350-500 | Premium, well-documented. Supports a wide range of engine PGNs |
| **Actisense EMU-1** | Analog engine senders | NMEA 2000 | $300-400 | Converts traditional analog senders (resistive, switch) to N2K PGNs |

**Which approach?** If your engine has J1939, use a gateway — you get ALL engine data
digitally. If your engine uses only analog senders (most sailboat diesels), either read
the alarm switches directly via Modbus DI (cheapest) or add an EMU-1/EngineBox to put
the data on N2K (richer data but more expensive).

---

## 4. Temperature monitoring

### Wireless (Bluetooth → Cerbo GX)

| Product | Range | Accuracy | Interface | ~Price | Notes |
|---|---|---|---|---|---|
| **RuuviTag Pro** | -40 to +85°C | ±0.1°C | Bluetooth → Cerbo GX | $30-40 ea | Battery lasts 2+ years. Ideal for fridge, freezer, engine room, ambient. The Cerbo collects them automatically via BLE and puts the data on MQTT for SignalK |

**Best approach for Victron owners:** RuuviTags are the easiest temperature sensors to
deploy — no wires, no Modbus configuration, just pair them with the Cerbo. Budget 4-6
tags (fridge, freezer, engine room, lazarette, cabin, cockpit).

### Wired (Modbus RTU or 1-Wire)

| Product | Range | Interface | ~Price | Notes |
|---|---|---|---|---|
| **DS18B20 (1-Wire)** | -55 to +125°C | ±0.5°C | 1-Wire → Pi GPIO | $2-5 ea | Cheap, waterproof probe versions available. Multiple sensors on one wire. Requires 1-Wire software on the Pi |
| **Modbus RTU temperature probe** (various; GAOTek, generic) | -40 to +250°C (varies) | ±0.3-0.5°C | RS-485 Modbus RTU | $30-80 | Gets its own Modbus address on the bus. No additional Pi configuration — the polling software reads it like any other module |

### Environmental (NMEA 2000)

| Product | Measures | Interface | ~Price | Notes |
|---|---|---|---|---|
| **Digital Yacht EnviroLINK** | Barometric pressure, temperature, humidity | NMEA 2000 (PGN 130310-130314) | $90-130 | Barometric trend is the key feature — rapid pressure change = weather coming. Accuracy: ±1.0 hPa, ±3% RH |
| **Quark-elec AS10** | Barometric pressure, temperature, humidity | NMEA 2000 | $60-90 | 3-in-1 sensor, low power (~4 mA from N2K bus). 1 Hz update rate |
| **LXNAV SmartSENSE** | Barometric pressure, temperature, humidity, dew point | Bluetooth + optional N2K | $100-150 | Battery-powered (72h). Also calculates dew point for condensation risk |

---

## 5. Current sensing (pump runtime monitoring)

A split-core Hall effect sensor clamps around a wire and outputs a current proportional to
the load. Wire the output to a Modbus AI channel. This is how you monitor bilge pumps,
refrigeration compressors, windlasses, and other loads WITHOUT breaking into the circuit.

| Product | Range | Output | ~Price | Notes |
|---|---|---|---|---|
| **YHDC HSTS016L** 20A split-core | 0-20A DC | 4-20mA (loop powered) | $8-15 | Non-invasive, clamp-on. Good for pump monitoring. Size the range to your load — don't use a 100A sensor on a 3A pump |
| **Accuenergy HAK series** | Up to 1000A | 4-20mA or 0-5V | $50-80 | Industrial grade, 0.5% accuracy. Overkill for most boat loads but available |
| **Aim Dynamics AIMHW** | Up to 400A | 4-20mA | ~$297 | IP68 waterproof. The only waterproof option found. Expensive; only needed for wet/bilge locations |
| **Oceancontrols WES-061** | 0-100A DC | 4-20mA (24V powered) | $30-50 | Split core, 25mm window. PLC/telemetry grade |

**Wiring:** Sensor 4-20mA output → Waveshare Analog Input channel. Set the AI module to
4-20mA input mode. Rule: "if channel > 4 mA for > 5 minutes, bilge pump running too long."

**Sizing:** Choose a sensor range close to your expected load. A 20A sensor reading a 2A
pump gives much better resolution than a 100A sensor on the same pump.

---

## 6. Tank levels

### Direct to Cerbo (Victron owners)

If you have a Victron Cerbo GX, the simplest path: the Cerbo has four dedicated tank
sender inputs that read standard resistive senders directly. No converter needed — just
wire the sender to the Cerbo's tank port and configure the resistance range and tank
capacity in the Cerbo's menu.

| Product | Resistance range | Interface | ~Price | Notes |
|---|---|---|---|---|
| **KUS USA reed-switch senders** | 240-30Ω (US standard) | Cerbo tank input | $30-60 | Available in 14", 16", and other lengths. Reed-switch design = stepped output (7-17% steps near the top). We use these on all four tanks — fuel P/S, fresh water P/S |

The Cerbo polls these at its own rate (typically 10-30 seconds). That's fine for
monitoring, but see the fast-polling alternative below if you need real-time updates
while actively filling.

### Existing senders (resistive → N2K converter)

If you don't have a Cerbo, or your Cerbo's four tank ports are full, you can convert
your existing resistive senders to NMEA 2000 with a standalone converter.

| Product | Input | Output | ~Price | Notes |
|---|---|---|---|---|
| **Navico fluid level sensor** | 33-240Ω resistive | NMEA 2000 | $100-150 | Reads your existing sender and puts it on the N2K bus. Supports fuel, water, waste, oil |
| **Actisense EMU-1** | Multiple analog + resistive | NMEA 2000 | $300-400 | Multi-input: handles engine senders AND tank senders. One device for everything analog → N2K |

### Resistive sender → Modbus AI (fast polling alternative)

> **Untested concept.** The circuit is straightforward and the math works on paper, but we
> haven't built this one yet. If you try it, the calibration curve (resistance → voltage →
> percentage) will need work. Treat this as "worth investigating" — especially if slow
> N2K polling frustrates you at the fuel dock.

The N2K converter path works for monitoring, but its update rate is slow — the Cerbo and
most N2K converters poll tank senders every 10-30 seconds. That's fine for watching levels
at anchor, but when you're standing at the fuel dock with a hose in the fill, you want
faster updates than that.

**The idea:** wire your resistive sender through a simple voltage divider into a
Waveshare Analog Input channel. Put a known resistor in series with the sender, feed it
a reference voltage, and read the midpoint voltage on the AI. The math converts voltage
back to resistance back to tank level. Your Modbus polling cycle is 1-2 seconds — that's
the update rate you'd get.

```
Reference voltage (5V or 12V)
        │
    [Known R]  (e.g., 240Ω for a 33-240Ω sender)
        │
        ├──────► Waveshare AI channel (reads voltage here)
        │
  [Tank sender]  (resistance varies with level)
        │
       GND
```

**When to use this path:**
- You're actively filling tanks and need fast updates
- You want tank levels in the Modbus polling loop (same latency as every other sensor)
- You don't have a Cerbo or N2K converter and don't want to buy one just for tanks

**When to use the N2K path:**
- The tank data is already on N2K (Cerbo, Navico converter)
- You don't need fast updates
- You want the data on N2K displays as well as in SignalK

### Ultrasonic (NMEA 2000, new installation)

| Product | Max depth | Output | ~Price | Notes |
|---|---|---|---|---|
| **Maretron TLM100** | 40" (1.02m) | NMEA 2000 | $250-350 | Non-contact ultrasonic. Calibrates for irregular tank shapes. Fuel, water, waste, oil. No probe inside the tank (mounts on top) |
| **Maretron TLM200** | 104" (2.64m) | NMEA 2000 | $300-450 | Deeper tanks. Same technology |

### Hydrostatic (NMEA 2000)

| Product | Input | Output | ~Price | Notes |
|---|---|---|---|---|
| **Oceanic Systems 4272** | Side-mount pressure | NMEA 2000 | $150-250 | ±2.5% accuracy. Mounts on the side of the tank. Range 0.5-4m depth |

---

## 7. Door, hatch, and security contacts

Magnetic reed switches are the standard. They're simple dry contacts — open when the
magnet separates (door opens), closed when together (door closed).

| Product | Type | Interface | ~Price | Notes |
|---|---|---|---|---|
| **Siren Marine SM-ACC-REED** | Wired magnetic reed | Dry contact | $18-27 | Marine-marketed. Screw + adhesive mount, 12" wire lead. Works with any DI. Can wire multiple in series for one input |
| **Generic IP67 reed switch** | Wired magnetic reed | Dry contact | $3-8 ea | Industrial reed switches from Mouser/Digikey work fine and come with published IP ratings. Look for IP67 minimum |
| **Aqualarm wireless security** | Wireless motion + door sensors + keypad | Wireless to Aqualarm panel | $200-400 (kit) | Self-contained security system. NOT wired to Modbus — it's an independent system |

**Wiring:** Reed switch → Modbus DI. Rule: "if companionway DI opens while alarm is armed,
alert." Multiple switches can be wired in series on one DI input (any one opening triggers
the alarm) or individually for per-hatch reporting.

**Hatch sealing:** For seacocks and hatch dogs, the magnet/switch alignment needs to
survive vibration and repeated opening. Test your mounting before committing — a magnet
that shifts 2mm can make the difference between reliable and false-alarm-prone.

---

## 8. Specialized sensors

### LPG / propane detection

If your boat has propane (common on galleys with gas stoves), a gas detector is required
by ABYC A-1. Most marine LPG detectors have a relay or solenoid output that shuts a gas
valve. Wire the alarm output to a Modbus DI for automation integration.

| Product | Interface | ~Price | Notes |
|---|---|---|---|
| **Fireboy-Xintex propane fume detector** | Relay output (for solenoid valve) + alarm | $100-200 | ABYC A-1 compliant. The relay drives the solenoid shut-off valve; tap the alarm signal for a Modbus DI |

### Aqualarm complete kits

Aqualarm sells pre-configured kits that bundle multiple sensors with a monitoring panel:

| Kit | Includes | ~Price | Notes |
|---|---|---|---|
| **Aqualarm 20006** (twin engine) | 5 detectors (bilge, cooling x2, fire, oil) + panel + horn | $400-600 | All-in-one for twin engines. Each detector's dry contact can also be tapped for Modbus DI integration |
| **Aqualarm automatic systems monitor** (single engine) | Similar, single engine | $250-400 | The entry-level Aqualarm package |

These kits give you standalone monitoring (panel + buzzer) out of the box, and each
sensor is a dry contact you can wire IN PARALLEL to a Modbus DI for automation.

---

## Integration patterns

### Pattern 1: Sensor → Modbus DI → rule → alert

The simplest pattern. A dry-contact sensor changes state, the rules engine reads the DI,
evaluates a condition, and sends a notification.

```
Float switch (NC) ──► Modbus Relay (D) DI channel 3
                              │
                              ▼
                       Rules engine: "if DI.3 open for > 3s, fire bilge_high rule"
                              │
                              ▼
                       Pushover notification to phone
```

### Pattern 2: Sensor → Modbus AI → rule → alert

For proportional measurements. A 4-20mA sensor feeds an analog channel; the rule
evaluates a threshold.

```
Hall effect CT (4-20mA) ──► Waveshare Analog Input CH 1
                                    │
                                    ▼
                             Rules engine: "if AI.1 > 4mA for > 300s, fire pump_runtime rule"
                                    │
                                    ▼
                             Pushover notification + log to history
```

### Pattern 3: Parallel tap (alarm panel + automation)

At Tier 3, each sensor feeds BOTH the standalone alarm panel AND the Modbus DI, so the
panel works independently of the Pi.

```
Float switch (NC) ──┬──► Alarm panel LED channel (standalone, Pi-free)
                    │
                    └──► Modbus DI (automation, rules, history, notification)
```

---

## Sensor shopping checklist

Before buying any sensor, verify:

- [ ] **Interface type** — dry contact, 4-20mA, resistive, N2K, or Bluetooth?
- [ ] **Fail-safe direction** — NC or NO? NC is preferred for alarm sensors (broken wire = alarm)
- [ ] **Mounting** — thread size, probe length, mounting surface, clearance
- [ ] **Environment** — IP rating for where it will live (IP67 minimum for bilge, engine room)
- [ ] **Power** — does it need a supply voltage, or is it passive (switch)?
- [ ] **Contact rating** — if driving a relay or buzzer directly, check the current rating
- [ ] **Thread / fitting** — match your engine, tank, or plumbing
- [ ] **Marine suitability** — tinned leads, corrosion-resistant body, vibration-resistant mount

---

## Sources

- [Aqualarm products (Fisheries Supply)](https://www.fisheriessupply.com/aqualarm) — full Aqualarm lineup
- [Aqualarm cooling water flow detectors](https://www.fisheriessupply.com/aqualarm-cooling-water-flow-detector/13209) — model 13209
- [Fireboy-Xintex two-zone panel (Boatersland)](https://www.boatersland.com/xintex-2-zone-fire-detection-alarm-panel.html) — fire detection
- [NASA EX-1 exhaust monitor (Seamark Nunn)](https://seamarknunn.com/products/nasa-marine-ex-1-exhaust-temperature-monitor-and-alarm) — exhaust temp
- [SMX exhaust thermal switch (SBMAR)](https://www.sbmar.com/product/smx-exhaust-heat-sensing-thermal-alarm-switch-180f/) — snap switch
- [Maretron TLM100 (Fisheries Supply)](https://shop.fieldandstream.com/products/maretron-tlm100-tank-level-monitor-40-depth-max-no-gas) — ultrasonic tank level
- [Digital Yacht EnviroLINK (Defender)](https://defender.com/en_us/digital-yacht-environlink-nmea-2000-barometer-marine-weather-station-zdigenvlink) — environmental sensor
- [Quark-elec AS10](https://www.quark-elec.com/product/as10-nmea-2000-3-in-1-environmental-sensor/) — environmental sensor
- [Siren Marine reed switch](https://reach.dog/shop/siren-marine-1/products/magnetic-reed-switch) — door/hatch contact
- [Oceancontrols WES-061](https://oceancontrols.com.au/WES-061.html) — split-core Hall effect sensor
- [Yanmar 4JH parts (FridayParts)](https://www.fridayparts.com/pressure-sensor-129a00-57100-for-yanmar-engine-3jh40-3tnv86cht-4jh110-4jh45-4jh57-4jh80) — engine sensor cross-ref
- [Navico fluid level sensor (Harbour Chandler)](https://www.harbourchandler.ca/products/lowrance-simrad-bg-fluid-level-sensor-33-240-ohm-to-nmea2000) — resistive-to-N2K
