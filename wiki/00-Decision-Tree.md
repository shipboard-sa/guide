<!--
Purpose: Requirements decision tree — help a new owner figure out what they want BEFORE
  looking at architecture or parts. This is Step 0 of the AI workflow.
Version: v1 (2026-11-04)
Scope: PUBLIC — the first page a new owner's AI session reads.
Architecture: Produces a requirements profile that feeds into 02-Architecture-Tiers.md.
  References automation-engine spec/00-Requirements.md for the worked example (Lee's answers).
Invocation: n/a (reference doc)
-->

# Step 0 — Decision Tree

Before you look at hardware, software, or wiring diagrams: figure out what you want.
Every answer below narrows the design space, and your AI session uses your answers to
pick the right [architecture tier](02-Architecture-Tiers.md) and generate a BOM
tailored to your boat.

There are no wrong answers. A weekender who wants dashboard monitoring is just as valid
as a liveaboard who wants full automation. The system scales — the tiers exist so you
build only what you need.

## The questions

### Q1. What level of automation do you want?

| Answer | What it means | Tier it leads to |
|---|---|---|
| **Monitor only** | See data on dashboards (temperatures, voltages, tank levels, wind, depth). No automated actions. Alarms are visual/audible on a screen — no physical alarm panel, no relay control. | Tier 1 |
| **Monitor + alert** | Everything above, plus phone notifications when something is wrong (bilge pump running, voltage high, anchor dragging). Still no automated actions — the boat tells you, you decide. | Tier 2 |
| **Monitor + alert + protect** | Everything above, plus the system can take protective action: kill the engine on cooling loss, stop charging on overvoltage, sound horns on intrusion. A physical alarm panel provides Pi-free backup. | Tier 3 |
| **Full system** (the reference build) | Everything above, plus maintenance tracking, passage planning, inventory, cockpit display, ship's log, anchor alarm. The complete system as documented in the reference repos. | Tier 4 |

### Q2. How do you use your boat?

| Answer | Design impact |
|---|---|
| **Coastal weekender** | Simpler setup. Cell coverage is reliable; notifications go straight to your phone. You're usually nearby. Battery monitoring matters less (shore power available). |
| **Coastal cruiser** | Moderate setup. You anchor out, so anchor alarm adds value. Battery monitoring matters (you run off the hook). Cell coverage is intermittent — consider satellite backup for alerts. |
| **Offshore passagemaker** | Full monitoring. You're away from help. Battery, bilge, engine, and weather monitoring are safety-critical. Satellite comms for alerts. Passage planning with weather routing. |
| **Liveaboard** (the reference build) | The boat IS your house. Everything matters: maintenance scheduling, provisioning, galley, history. You want the full system because you use it every day. |

### Q3. What is your budget?

| Tier | Approximate cost | What you get |
|---|---|---|
| **~$200** | Raspberry Pi + SignalK + existing NMEA instruments | Dashboards, history, basic alerts |
| **~$800** | Above + Modbus I/O modules + phone notifications | Sensor monitoring, alarm notifications |
| **~$2,000** | Above + relay pool + alarm panel + anchor alarm box | Automated protection, physical alarms |
| **~$5,000+** | The full reference build | Everything including custom PCB panel, passage planning, maintenance system |

These are hardware costs only. Software is free (open source). Your time is the biggest
investment — the reference build took ~6 months of evenings and weekends, with an AI
assistant doing the software and an experienced electrical engineer doing the wiring.

### Q4. How technical are you?

| Answer | What it means for your build |
|---|---|
| **I can install a chart plotter and run wires** | You can do Tiers 1-2 yourself. Tier 3 wiring should be reviewed by a marine electrician. |
| **I'm comfortable with 12/24V DC wiring and crimping** | You can do Tiers 1-3 yourself with the [Before You Build](05-Before-You-Build.md) guide. |
| **I'm an electrician / electronics hobbyist** | You can do the full build. The reference repos have everything you need. |
| **I write code and solder** | You can modify everything — firmware, PCB, rules engine, apps. The repos are your playground. |

### Q5. What is your risk tolerance for unattended operation?

| Answer | Design impact |
|---|---|
| **The system should never act on its own** | Monitor + alert only (Tiers 1-2). Every action requires a human. |
| **Protective actions are OK if they fail safe** | Tier 3: the system can stop an engine or cut charging, but the default (dead system) leaves the boat running safely. Every automated output has a manual override in copper. |
| **I trust automation for routine operations** | Tier 3+: automated blower control, scheduled pump runs, mode-based behavior. The reference build's philosophy: the boat works WITH you, never FOR you. Manual override always. |

### Q6. Do you anchor out?

| Answer | Design impact |
|---|---|
| **Rarely — I mostly dock** | Skip the standalone anchor alarm. Your chartplotter's built-in anchor watch is sufficient for occasional use. |
| **Regularly** | Consider the standalone anchor alarm (anchor-alarm repo). The reference build uses a dedicated ESP32 touchscreen with GPS circle-fit — more reliable than a chartplotter anchor alarm because it's independent (no shared processor, no chart rendering competing for attention, survives a plotter crash). |
| **I live on the hook** | The standalone anchor alarm is strongly recommended. The reference build also integrates it with the alarm panel (physical buzzer, horn) and night-watch check-in. |

## Your requirements profile

After answering Q1-Q6, you have a requirements profile:

```
Automation level: [Monitor / Monitor+Alert / Monitor+Alert+Protect / Full]
Usage pattern:    [Weekender / Coastal / Offshore / Liveaboard]
Budget:           [$200 / $800 / $2,000 / $5,000+]
Technical level:  [Basic / DC-comfortable / Electrician / Coder]
Risk tolerance:   [Human-only / Fail-safe protection / Routine automation]
Anchoring:        [Dock / Regular / Liveaboard]
```

Hand this to your AI session. It maps directly to an
[architecture tier](02-Architecture-Tiers.md) and a filtered
[BOM](04-Reference-BOM.md).

## Worked example: the reference build

The SV Unladen Swallow 2 build answered: Full system / Liveaboard / $5,000+ /
Electrician+Coder / Fail-safe protection / Liveaboard. Lee's full requirements
are in `automation-engine/spec/00-Requirements.md` — 19 approved requirements
(R-01 through R-09 functional, N-01 through N-10 non-functional, DR-01 through
DR-04 design rules). You do not need all of them. Your AI session will help you
pick the subset that matches your profile.
