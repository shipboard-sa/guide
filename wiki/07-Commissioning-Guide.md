<!--
Purpose: How to commission and test the finished system — Step 7 of the AI workflow.
Version: v2 (2026-11-04) — DRAFT. Derived from automation-engine spec/12-Operations.md v3.1
  and wiring-io/wiki/Circuit-Register.md v1.2.
Scope: PUBLIC — generic commissioning methodology.
Architecture: References wiring-io for the worked example, automation-engine for the OQ
  pattern and health-monitoring architecture.
Invocation: n/a (reference doc)
-->

# Step 7 — Commissioning Guide

Commissioning is how your wiring and rules become a trusted system. The pattern:
**ring out → prove → record → accept → go live**, one circuit at a time. Nothing
goes live without evidence.

This page teaches the methodology. Your AI session uses it to walk you through
commissioning each circuit and rule in your build.

---

## 1. Circuit states

Every circuit in your system has a status. It moves forward only with evidence.

```
drawn → wired → proven → live → (retired)
```

| State | Meaning | What moves it forward |
|---|---|---|
| **drawn** | Wiring sheet and slide exist, nothing wired | You pick up the crimper |
| **wired** | Copper landed on terminals | Ring-out (continuity and isolation test) |
| **proven** | Tested aboard for its function, with evidence | Your acceptance (you've seen it work) |
| **live** | Rules may read or drive this circuit | Time under observation with no false alarms |
| **retired** | Removed or replaced | A decision recorded with reason |

The matching state in `io_master.yml`: `unverified` (drawn/wired), `verified`
(proven), `live`. The engine enforces this — a live rule that reads or drives a
non-live circuit is rejected at load.

---

## 2. The ring-out

Before you apply power to any new circuit, verify every wire with a multimeter.

### What you check

With the circuit **de-energized** (battery disconnected or fuse pulled):

| Check | How | Pass |
|---|---|---|
| **Continuity** | Meter on each wire, end to end | Reads near 0 Ω on every conductor |
| **No shorts to ground** | Each conductor to the ground bus | Reads OL (open) on every conductor |
| **No shorts between conductors** | Each conductor to every other | Reads OL between every pair |
| **Polarity** | Trace + and − to their terminals | Matches the wiring sheet |

### What it catches

- Missed landings (a wire in the wrong terminal)
- Damaged insulation (conductor touching the chassis or another wire)
- Swapped polarity (reversed + and −)
- Broken conductors (a crimp that didn't bite)

### Recording the result

"Ring-out complete" means **every reading matched the prediction.** If something
didn't match, that's a finding — record what you expected, what you got, and what
you did about it. Then re-test.

---

## 3. The proof test

After ring-out, apply power and verify the circuit does what the wiring sheet says.

### For a sensor input (DI)

1. **Trigger the sensor** — lift the float switch, press the reed magnet, apply heat
   to the detector, or use the sensor's built-in test mode
2. **Verify the signal** — the Modbus module's DI changes state (check the engine
   status page or probe the register directly)
3. **Verify the panel** — if this sensor drives a panel lamp, the lamp lights
4. **Verify the rule** — if a dry_run rule reads this signal, check the engine log
   for "would fire"

### For a relay output (DO)

1. **Command the relay** — use the engine's probe command or the test UI to pulse
   the relay for 2 seconds
2. **Verify the load** — the connected device responds (motor runs, solenoid clicks,
   contactor engages)
3. **Verify the return** — release the relay, verify the load returns to its default
   state (fail-safe: relays are normally de-energized)

### For an end-to-end protection circuit (Tier 3+)

The full chain from sensor to action:

1. **Force the sensor condition** safely (test signal, not a real emergency)
2. **Verify the alarm path** — panel lamp, buzzer/horn, Pi notification, phone push
3. **Verify the protection action** — the relay fires, the load responds
4. **Verify the manual override** — flip the copper override switch, confirm it
   overrides the automation
5. **Verify recovery** — acknowledge the alarm, reset, verify normal operation returns

### Evidence format

Each proof test gets a finding entry:

```
F-nnn  (date)  Circuit/point tested
  What was tested: [description of the test]
  What happened: [what you observed]
  Result: PASS / FAIL / PARTIAL
  Evidence: [photo, log entry, meter reading]
```

A PASS means the circuit behaves exactly as the wiring sheet says. PARTIAL means
it works but something was unexpected. FAIL means it didn't work — record what went
wrong and what you'll fix.

---

## 4. Per-tier commissioning checklist

### Tier 1 — Dashboard only

| Check | How to verify | Done when |
|---|---|---|
| Pi boots and stays up | SSH in, `uptime` shows hours not minutes | Stable for 24 hours |
| SignalK receives NMEA data | SignalK admin → Data Browser shows live values | All your instruments appear |
| Grafana dashboards load | Open each dashboard, verify data is current | No "No Data" panels |
| History records | Check yesterday's data exists in Grafana | Time range picker shows last 7 days |

### Tier 2 — Monitoring + alerts

Everything in Tier 1, plus:

| Check | How to verify | Done when |
|---|---|---|
| Each Modbus module answers | Engine status page or `probe modbus --scan` | Every module in io_master responds |
| Each sensor reads correctly | Compare DI/AI values against physical state | All sensors agree with reality |
| dry_run rules evaluate | Engine log shows "would fire" on test conditions | Every rule in your `rules.yml` has been seen to evaluate |
| Notifications arrive | Trigger a test NOTICE, verify phone push | Notification received within 30 seconds |
| `stale` rules fire on sensor removal | Disconnect one sensor, verify stale alarm | Alert within the configured stale timeout |

### Tier 3 — Protection + alarm panel

Everything in Tier 2, plus:

| Check | How to verify | Done when |
|---|---|---|
| Each panel lamp lights | Ground-test each panel input (or use the sensor) | All lamps light on the correct condition |
| Horn/buzzer sounds | Trigger an EMERGENCY alarm | Horn sounds, stops on acknowledge |
| Each protection relay operates its load | Probe/pulse each relay, verify load response | Every relay proven with its load |
| Manual overrides work | Flip each copper override switch during an alarm | Override stops the automated action |
| End-to-end protection test | Force sensor → alarm → relay → load → override → recovery | Every protection circuit proven as a chain |
| Heartbeat LED | Kill the engine process, verify the panel shows it | Heartbeat LED goes dark within the configured timeout |

### Tier 4 — Full system

Everything in Tier 3, plus:

| Check | How to verify | Done when |
|---|---|---|
| mx-inv serves on the network | Open in a browser, navigate to each section | All pages load, data is current |
| Maintenance tasks generate | Check that scheduled tasks appear at their intervals | First round of tasks has appeared |
| Cockpit display shows live data | Walk to the cockpit, verify current readings | All configured values display |
| Ship's log records events | Trigger an alarm, verify it appears in the log | Event appears with correct timestamp and details |
| Anchor alarm | Set anchor, test drag detection | Alarm fires on simulated drag |
| 2-week soak | Run the full system for 14 days at the dock | No false alarms, no missed real events, no crashes |

---

## 5. The dry_run → live promotion

Rules follow the same evidence-based progression as circuits.

### Promotion steps

1. **Deploy the rule as `dry_run`.** It evaluates on real data but takes no action.

2. **Wait for the real condition.** Don't just simulate — wait for the actual
   environmental condition to occur (the bilge pump actually runs, the battery
   actually dips, the wind actually picks up). This takes days or weeks. That's fine.

3. **Simulate the condition.** Use `simulate io.signal=value for 60s` to force
   conditions you can't wait for. Every event from simulated input is tagged
   `SIMULATED` — you can distinguish real from test in the logs.

4. **Verify the evidence.** For each rule, you should have seen:
   - At least one real evaluation (from watching)
   - At least one simulated trigger (from testing)
   - The correct message text
   - No unexpected side effects

5. **Check the circuit states.** The rule can only go live if every circuit it reads
   or drives is itself `live` in io_master. This is enforced — the engine rejects a
   live rule on a non-live circuit.

6. **Promote.** Change `mode: live` in `rules.yml` and reload the engine. **One rule
   at a time.** If three rules go live together and something goes wrong, you don't
   know which one caused it.

### Simulation guardrails

- A simulated signal can never drive a `PROTECT` relay live — the engine forces
  dry_run for PROTECT actions during simulation
- A test session records relay states at start and restores them at end
- Test-activated lockouts block only other test actions, never real protection
- Test sessions auto-expire (default 1 minute, maximum 1 hour)

---

## 6. Health monitoring — what watches the watcher

Once your system is live, you need to know when something breaks.

### The health stack (build from the bottom up)

| Layer | What it does | Your tier |
|---|---|---|
| **Engine heartbeat** | The rules engine reports healthy when it's running, polling, and all required signals are fresh | Tier 2+ |
| **Container healthchecks** | Docker checks each container is responding to its health endpoint | Tier 2+ |
| **Watchdog** | A separate process restarts unhealthy containers (bounded — gives up after 3 tries/hour, tells you) | Tier 3+ |
| **Alarm panel heartbeat** | A hardwired LED that goes dark when the Pi stops sending its heartbeat signal | Tier 3 |
| **Phone notifications** | Push notifications for alarms, watchdog actions, and health changes | Tier 2+ |
| **Smoke checks** | A read-only script that verifies every service is responding | All tiers |

### The key principle

The alarm panel is primary. It works with no Pi, no network, no software. The Pi
adds intelligence (rules, history, notifications), but when the Pi dies, the copper
layer keeps working. Your commissioning must verify both paths:

1. **Pi alive:** sensor → Modbus → engine → alarm + relay + notification
2. **Pi dead:** sensor → hardwired panel circuit → lamp + buzzer (Tier 3)

### What to monitor in steady state

| Signal | What it means if it fires | Action |
|---|---|---|
| Heartbeat LED dark | Pi or engine process down | Check the Pi, check Docker, check the engine logs |
| Module stale alarm | A Modbus module stopped responding | Check that module's power, check the bus wiring at that point |
| Sensor stale alarm | A specific sensor stopped reporting | Check that sensor, check its wiring |
| Watchdog notification | A container was unhealthy and restarted | Read the container logs — the cause is usually external (network, power, hardware) |
| Watchdog gave-up alarm | 3 restarts didn't fix it — human needed | The problem is bigger than a restart. Read logs, fix the cause manually |

---

## 7. The circuit register — your commissioning tracker

Track every circuit in one table. This is the single place to look when someone asks
"is X working?"

### Format

```markdown
| Circuit | Status | Proof (date) | Channels | What is left |
|---|---|---|---|---|
| Bilge pump monitoring | proven 2027-01-15 (F-012) | float triggered → DI read → alarm → notification | S1-DI2 live | Go-live after 1 week dry_run |
| Engine kill | wired, not proven | Ring-out clean (F-015) | S3-R1 unverified | End-to-end test with engine running |
| Shore power detection | live | Ground test 2027-01-20 (F-018), rule promoted 2027-02-01 | S2-DI5 live | None |
```

### Column rules

| Column | What goes in it |
|---|---|
| **Circuit** | The circuit name from your wiring sheet |
| **Status** | drawn / wired / proven / live / retired |
| **Proof** | The finding ID and date of the test that moved it to this status |
| **Channels** | The io_master channels with their current state |
| **What is left** | What must happen before the next status. Empty = done |

### Update rules

- Update the register in the **same session** as any wiring job, proof test, or
  state change
- Re-check every row against io_master at each checkpoint
- If a row conflicts with io_master, io_master wins — fix the register

---

## 8. The findings register

Every test result, observation, and anomaly gets a numbered finding.

### Format

```
F-001  2027-01-10  Salon bilge float switch ground test
  Meter on S1-DI2 with float lifted: reads TRUE. Float down: FALSE.
  Panel lamp AL-4 lights when float is up.
  Result: PASS. Circuit proven.

F-002  2027-01-10  Salon bilge float — slow return
  Float switch sticks for ~2 seconds after lowering.
  Probably debris in the well. Cleaned. Re-tested: returns in <0.5 s.
  Result: PASS after cleaning.

F-003  2027-01-11  Bow high water — wrong DI
  Expected S1-DI1, got S1-DI3. Wires landed on the wrong terminals.
  Swapped at the module. Re-tested: correct.
  Result: PASS after correction. Wiring sheet updated.
```

### What to record

- Every ring-out (pass or fail)
- Every proof test (pass, partial, or fail)
- Every anomaly during testing (unexpected readings, slow responses, intermittent
  behavior)
- Every correction and its re-test
- Every decision to promote a rule to live

### Why it matters

Six months from now, when something behaves oddly, the findings register tells you:
- Whether this circuit was ever fully tested
- What was found during testing
- What was corrected
- When it went live

Without the register, you're debugging from scratch. With it, you know the history.

---

## 9. What "done" looks like

### Per tier

| Tier | Done when |
|---|---|
| **1** | Dashboards show all your instruments. History is recording. You can see yesterday's data |
| **2** | Every sensor in your survey has a rule. All dry_run rules promoted to live. Notifications working. No false alarms for 1 week |
| **3** | Every protection circuit proven end-to-end. Alarm panel verified (every lamp, the buzzer, the heartbeat). Manual overrides tested. No false alarms for 2 weeks |
| **4** | Full system operating for 2 weeks with no false alarms. Maintenance tasks generating on schedule. Ship's log recording. Crew can operate without your AI session |

### The acceptance question

For each circuit and rule, the question is:

> "If I leave the boat at the dock and come back in a week, will this circuit/rule
> do the right thing if the real condition occurs, and will it NOT do anything if
> the condition doesn't occur?"

If you can't answer yes to both parts, it's not done.

### The handover test

Your system is ready for unsupervised operation when:

1. **Every circuit is live** — no unverified channels that rules depend on
2. **Every rule is live** — no dry_run rules for conditions that matter
3. **The alarm panel works standalone** — kill the Pi, verify the copper layer still
   alarms on every sensor condition (Tier 3)
4. **Recovery is documented** — for each known failure mode (Pi down, module dead,
   sensor dead, network dead), you've written what happens and what to do
5. **The crew understands the overrides** — every copper override switch is labeled,
   and the crew knows what each one does

---

## 10. Ongoing operations

After commissioning, the system needs light maintenance.

### Regular checks

| What | How often | How |
|---|---|---|
| Smoke check | Every deploy, or monthly | Run the read-only verify script — it checks every service is responding |
| Check for stale findings | Monthly | Review the findings register for items marked "not recorded" or "assumed" |
| Review false alarms | After any false alarm | Adjust the threshold, debounce (`for:`), or hysteresis (`clear_at`) |
| Verify backup | Monthly | Confirm the backup ran and a restore produces valid data |
| Check sensor health | Quarterly | Walk the boat, verify each sensor is physically intact and its reading matches reality |

### Adding a new circuit later

The same process applies, just for one circuit:

1. Draw the wiring sheet
2. Wire it
3. Ring out
4. Prove it
5. Add the rule (dry_run)
6. Watch and simulate
7. Promote to live
8. Update the circuit register

Your AI session handles the rule writing (Step 6) and can walk you through the
testing. You handle the crimping.

---

## Reference

- `wiring-io/wiki/Circuit-Register.md` — the reference build's circuit register
  (every W-sheet and panel point with status, proof, and what's left)
- `automation-engine/spec/12-Operations.md` — the reference build's full operations
  manual (health monitoring, routine changes, known failures, testing)
- `automation-engine/docs/Commissioning-Log.md` — the reference build's
  chronological commissioning evidence
- `automation-engine/spec/03-Findings.md` — the reference build's findings register
