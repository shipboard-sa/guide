<!--
Purpose: Buyer's guide to the Waveshare Modbus RTU module lineup and competing ecosystems.
Version: v1 (2026-11-04)
Scope: PUBLIC — hardware selection guide for new owners choosing I/O modules.
Architecture: References wiring-io/wiki/IO-and-Relays.md for the reference build's actual usage.
Invocation: n/a (reference doc)
-->

# Waveshare Modbus RTU Module Buyer's Guide

Waveshare makes the I/O modules the reference build runs on. This page describes every
module in the lineup, explains when to use each one, and surveys competing ecosystems so
you can make an informed choice.

---

## Why Waveshare

- **Price.** An 8-channel relay+DI module costs ~$35-50. Industrial equivalents (Advantech,
  Moxa) start at $200+ per module.
- **Availability.** Amazon, Pimoroni, RobotShop, and dozens of other retailers stock them.
  Lead time is 1-2 weeks, not 6-8.
- **Simplicity.** Standard Modbus RTU over RS-485, 9600 8N1. No proprietary software, no
  license fees, no cloud account.
- **Adequate for the job.** These modules run at residential/light-industrial specs (ABS
  enclosures, 0-55 °C rated, opto-isolated inputs). On a boat behind a master breaker,
  fused and DIN-mounted, they work. The reference build has run them for a full refit cycle
  with no failures.

**The trade-off:** no marine certifications (no UL Marine, no ABYC compliance stamp). Your
installation must comply — the modules themselves are just Modbus slaves on a bus.

---

## The Waveshare module lineup

### Quick comparison

| Module | Relays | DI | DO | AI | Form factor | ~Price | Role |
|---|---|---|---|---|---|---|---|
| **Relay (D) 8ch+8DI** | 8 (1NO+1NC, 10A) | 8 (NPN, isolated) | — | — | larger | $35-50 | Foundation: relays + alarm inputs in one box |
| **IO 8CH** | — | 8 (NPN, pull-to-ground) | 8 (open-drain, 500 mA) | — | **smaller** | $25-35 | Scaling: more DIs + low-current DOs, less rail |
| **Relay 8ch** | 8 (1NO+1NC, 10A) | — | — | — | medium | $30-45 | Relay-only when you need outputs, not inputs |
| **Relay 8ch (E)** | 8 (1NO+1NC, 10A) | — | — | — | medium | $35-50 | Relay 8ch variant: higher current rating, improved isolation |
| **Analog Input 8CH** | — | — | — | 8 (12-bit, 0-5V or 4-20mA) | medium | $30-45 | Current sensing, tank levels, analog sensors |
| **Relay 16ch** | 16 (1NO, 10A) | — | — | — | large | $55-80 | Mid-scale relay needs |
| **Relay 30ch** | 30 (1NO, 10A) | — | — | — | large | $80-120 | Cost-effective bulk relays for large builds |
| **Relay 32ch** | 32 (1NO, 10A) | — | — | — | large | $90-130 | Largest single-enclosure option, opto-isolated |

All modules: Modbus RTU, RS-485, 7-36V supply (except 5V variants of some relay modules).
DIN rail or wall mount. Each gets a unique Modbus address (1-247).

### Module details

#### Relay (D) 8ch + 8 DI — the entry point

This is the minimum Waveshare module. One box gives you 8 relays (for protection outputs
like engine kills and load shedding) AND 8 digital inputs (for alarm sensors like float
switches, door contacts, and fire detectors). At Tier 2, this is your first module.

- **Relays:** 1NO+1NC per channel, rated 10A at 250VAC or 30VDC. Non-latching (drops on
  power loss — choose fail-safe direction per circuit).
- **Digital inputs:** NPN pull-to-ground, opto-isolated. COM on +24V. A sensor grounds the
  input to signal. Works with any dry-contact sensor: float switches, reed switches,
  pressure switches, thermal snap switches.
- **Indicator LEDs:** One per relay, one per DI.
- **Bus:** RS-485, 9600 8N1, configurable address.

#### IO 8CH — the scaling module

When you need more digital inputs and your DI count outgrows the Relay (D), the IO 8CH
adds 8 DI + 8 DO in a smaller footprint. The DOs are NOT relays — they are open-drain
transistor outputs that sink up to 500 mA to ground. That's enough for LED drivers, logic
signals, or small solenoids, but not for switching pumps or motors.

- **DI:** 8 channels, NPN pull-to-ground, same interface as the Relay (D) DIs.
- **DO:** 8 channels, open-drain, 500 mA max per channel. Sink-to-ground only.
- **Form factor:** Noticeably smaller than the Relay (D). If rail space is limited, this
  is why you use IO modules for inputs instead of adding another Relay (D).

#### Relay 8ch — relay-only

Eight relays, no inputs. Use when you need relay outputs but already have enough DI
channels from your Relay (D) or IO modules. The reference build uses one as S6 — a
dedicated module for engine kill relays, physically separated from S3 (the Relay D) so
that a single module failure doesn't take out both kill circuits.

#### Analog Input 8CH — current and voltage sensing

Eight analog channels, 12-bit ADC, configurable for 0-5V or 4-20mA input ranges. Use
for:

- **Pump runtime monitoring:** Clamp a split-core Hall effect sensor around a pump's supply
  wire, read the current on an analog channel. Rule: "if bilge pump current > 0 for > 5
  minutes, alert."
- **Tank levels:** Some tank senders output a variable voltage or current. Read it here
  instead of (or in addition to) the N2K bus.
- **Analog sensors:** Anything that outputs 0-5V or 4-20mA — temperature transmitters,
  pressure transmitters, flow meters.

#### Relay 30ch / 32ch — bulk relays

If you're building a large system (20+ relay channels), these put 30 or 32 relays in one
enclosure at a much lower per-channel cost than individual 8-channel modules. Trade-off:
all channels share one module, so you lose the redundancy of splitting safety-critical
relays across separate hardware.

**Recommended approach for large builds:** Use a 30ch for general automation relays and a
separate 8ch for safety-critical kills (engine stop, charge disconnect). That way a module
failure can't take out both safety and comfort functions.

---

## Bus architecture

```
Pi + RS485 CAN HAT  (or Ethernet Gateway for Tier 3+)
        │
        │  RS-485 bus (shielded twisted pair, 120R at each end)
        │
  ┌─────┴──────┬─────────────┬────────────────┐
  │            │             │                │
Relay (D)   IO 8CH      Relay 8ch     Analog Input
addr 3      addr 4       addr 1         addr 5
(S3)        (S1)         (S6)           (S5)
```

Every module gets a unique address. The bus is daisy-chained — one pair of wires runs from
module to module. The reference build's chain: Gateway → S6 → S1 → S2 → S3, with the
Analog Input (S5) not yet wired at time of writing.

**Tier 2 bus (HAT as master):** The Pi polls every module directly over the HAT's RS-485
port. Simple, cheap, one point of failure (the Pi).

**Tier 3 bus (Ethernet gateway + HAT as cold spare):** The Waveshare RS485 TO POE ETH (B)
sits at the head of the bus. The Pi polls over Ethernet (tcp://gateway-ip:502). If the
gateway fails, swap to the HAT (it's still wired to the bus). If the Pi fails, the alarm
panel is Pi-independent and keeps working.

---

## Competing ecosystems

### Industrial Modbus (Advantech ADAM, Moxa ioLogik)

| | Waveshare | Advantech ADAM-4000/6000 | Moxa ioLogik E1200 |
|---|---|---|---|
| **Protocol** | Modbus RTU (RS-485) | ADAM-4000: RTU; ADAM-6000: TCP (Ethernet) | Modbus TCP (Ethernet) |
| **Price per 8-ch relay** | ~$35-50 | ~$200-400 | ~$250-500 |
| **Enclosure** | ABS, DIN rail | Metal, DIN rail | Metal, DIN rail |
| **Certifications** | CE | UL, CE, FCC, marine options | UL, CE, marine options |
| **Operating temp** | 0 to 55 °C (typical) | -10 to 70 °C (industrial) | -10 to 60 °C |
| **Software** | None (Modbus registers, read the datasheet) | ADAM.NET utility, web config | IOxpress utility, web config, SNMP |
| **Isolation** | Opto-isolation on inputs | Transformer + opto | Transformer + opto |
| **Best for** | Budget builds, makers, prototype-to-production | Industrial spec, regulatory compliance | Industrial spec, Ethernet-first networks |

**Bottom line:** If your insurer or surveyor requires UL-listed or marine-certified I/O
modules, Advantech or Moxa is the path. If you're building to learn and your modules sit
behind your own fuse panel, Waveshare does the same job at 10-20% of the cost.

### Raspberry Pi HATs and direct-attach I/O

| Product | Channels | Interface | ~Price | Notes |
|---|---|---|---|---|
| Sequent Microsystems Mega-IO | 8 DI + 4 relay + 8 AI (10-bit) | I²C (Pi HAT) | $50-60 | All-in-one HAT but limited to one per stack without address tricks |
| Pimoroni Automation HAT Mini | 3 relay + 3 analog + 3 DI | SPI (Pi HAT) | $25-35 | Small channel count; good for a quick start but doesn't scale |
| Waveshare High-Precision AD/DA Board | 8 AI (24-bit) + 2 AO | SPI (Pi HAT) | $25-35 | High-resolution analog only; no relays, no DI |

**Why Modbus wins for boats:** HATs give you a few channels cheaply, but they're physically
on the Pi and can't be extended. Modbus modules live on a bus — run a cable to the engine
room, mount modules there, and add more as you grow. For a boat where sensors are
distributed across 40+ feet, Modbus is the right architecture. HATs are fine for the helm
station's local I/O.

### Chinese Modbus clones (AliExpress / Spotpear / no-name)

You'll find RS-485 Modbus relay modules on AliExpress for $15-25. Some are perfectly
functional; some have wrong register maps, no documentation, or inconsistent quality.

**If you go this route:**
- Buy one first and test the register map against the datasheet.
- Check that the relay rating is real (some budget modules derate badly at DC voltages).
- Verify isolation — some cheap modules share grounds between the relay side and the logic
  side, which defeats the purpose.
- Waveshare's documentation and community support are the actual value add over no-name
  modules.

---

## Decision flowchart

```
Do you need relays AND digital inputs?
├── Yes → Relay (D) 8ch+8DI  (the foundation)
└── No
    ├── Need relays only?
    │   ├── < 10 channels → Relay 8ch
    │   ├── 10-20 channels → Relay 16ch or 2x Relay 8ch
    │   └── 20+ channels → Relay 30ch + separate 8ch for safety kills
    ├── Need more DI + low-current DO?
    │   └── IO 8CH  (smaller, cheaper, 500mA DOs)
    └── Need analog sensing?
        └── Analog Input 8CH
```

**For a typical Tier 2 start:** Relay (D) + Analog Input = 8 relays, 8 DI, 8 AI. That's
your foundation.

**For Tier 3 scaling:** Add IO 8CH modules for more DIs, a second Relay 8ch for redundant
kills, and the Ethernet gateway.

---

## Practical notes

1. **Address planning.** Decide your addresses before you buy. The reference build uses S1
   through S6 (with gaps for future modules). Write it down, because you'll configure each
   module's address via its DIP switches or holding registers.

2. **Power budget.** Each relay coil draws current. An 8-channel module with all 8 relays
   energized pulls ~1A at 12V. Plan your fuse block and supply accordingly.

3. **Bus length.** RS-485 is rated for 1200m, but on a boat 50-100 feet of cable is
   typical. Use shielded twisted pair. Shield grounded at ONE end only.

4. **Termination.** 120R resistor at each end of the bus. The HAT usually has one built in
   (check the dip switch). Put the second at the last module on the chain.

5. **Module failure.** Non-latching relays drop to their de-energized state on power loss
   or module failure. Choose which state is fail-safe for each circuit BEFORE you wire it.
   Engine kill: NC in series (dead relay = engine stays running). Charge disconnect: NC
   energize-to-disconnect (dead relay = loads stay connected).

6. **Where to buy.** Waveshare's own store (waveshare.com) and authorized resellers
   (Pimoroni, RobotShop) are typically 15-25% cheaper than Amazon for the same modules.
   Amazon's advantage is speed (2-day shipping vs. 1-2 weeks). If your build timeline
   allows it, order direct and save the markup. If you need it Thursday, Amazon works.

---

## Sources

- [Waveshare Modbus RTU Relay (D)](https://waveshare.com/modbus-rtu-relay-d.htm) — official product page
- [Waveshare IO 8CH](https://www.waveshare.com/modbus-rtu-io-8ch.htm) — official product page
- [Advantech ADAM-6060 (RS-Online)](https://uk.rs-online.com/web/p/plc-io-modules/0718825) — industrial comparison
- [Waveshare 32-Ch relay (Botland)](https://botland.store/relay-modules/24985-relay-module-32-channels-with-optoisolation-10a250vac30vdc-contacts-7v-36v-coil-modbus-rs485-waveshare-25140-5904422385750.html) — bulk relay option
