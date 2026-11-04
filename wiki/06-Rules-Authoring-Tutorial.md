<!--
Purpose: Rules language reference — the format an AI session uses to write automation rules.
  Step 6 of the AI workflow.
Version: v2 (2026-11-04) — DRAFT. Derived from automation-engine spec/04-Interfaces/rules-schema.md v11.
Scope: PUBLIC — language reference for the rules-as-data format.
Architecture: References rules-schema.md for the canonical spec. This page is a consumable
  reference that an AI session reads to write correct rules for a new owner's system.
Invocation: n/a (reference doc)
-->

# Step 6 — Rules Language Reference

Rules are YAML data, not code. The engine is generic Python — it loads `rules.yml`,
evaluates every row each tick, and acts. Adding an alarm means adding a row and
reloading. No code change.

This page is the language reference. Read it top to bottom once, then use it as a
lookup when writing rules.

---

## One rule, fully annotated

```yaml
- id: WF-201
  purpose: "Catch raw-water cooling failure before the engine overheats"
  when: {sk.yanmar_rpm: {above: 500, for: 30s}}
  if:   {io.yanmar_raw_water_flow: {is: false, for: 15s}}
  mode: dry_run
  do:
    alarm: {severity: EMERGENCY}
    message: "EMERGENCY raw-water flow lost — check strainer / impeller; engine stops in 15 s unless acked"
    after_unacked: {in: 15s, do: {set: {io.yanmar_stop_flow_smoke: on}}}
  inhibited_by: [WF-007]
  clear: manual
  note: "copper override: helm AUTO STOP OFF rocker"
```

| Key | What it does |
|---|---|
| `id` | Unique identifier (WF-nnn by convention) |
| `purpose` | One line: what bad thing this catches. Documentation only |
| `when` | Arming condition — the rule is inert unless this is true |
| `if` | Trigger condition — evaluated only while armed |
| `mode` | `dry_run` (log only) or `live` (act). Every rule starts dry_run |
| `do` | One or more actions (alarm, message, set, etc.) |
| `inhibited_by` | List of rule IDs that suppress this rule when active |
| `clear` | `auto` (default — clears when condition clears), `on_ack`, or `manual` |
| `note` | Free text — usually names the copper override |
| `record` | Where events go: `handover` / `log` / `audit` (see Actions) |

---

## 1. Signals — what a rule can read

A signal is a named value the engine knows about. Signals come from five sources.

| Prefix | Source | Example |
|---|---|---|
| `io.*` | Modbus I/O point (from `io_master.yml`) | `io.high_water_salon` |
| `sk.*` | SignalK path (NMEA 2000, instruments) | `sk.propulsion.main.revolutions` |
| `cerbo.*` | Victron Cerbo GX via MQTT | `cerbo.house_soc` |
| `ssm.*` | Battery pack / cell data | `ssm.any.cell_max_v` |
| `net.*` | A URL that must answer (health check) | `net.anchor_box` |
| `host.*` | A number read from a file on the host | `host.cpu_temp` |
| `cam.*` | Camera AI person detection | `cam.cockpit_person` |

**Special signals:**

| Signal | What it is |
|---|---|
| `mode` | Boat mode: `DOCK`, `ANCHOR`, `MOTOR`, `SAIL`, `AWAY` |
| `active.<rule_id>` | True while another rule is active |
| `ack.<rule_id>` | True when a rule has been acknowledged |
| `modbus.<module>` | True while a Modbus module is answering |
| `night_watch` | The crew's ARM tap (watchkeeper timer) |
| `fence.armed` / `fence.distance` | Slip fence state and meters from dock center |

### Signal resolution

A signal that has never reported, or whose last reading is older than its `max_age`,
reads `UNKNOWN`. **A condition on UNKNOWN is not met.** This means: if a sensor dies,
rules that depend on it stop firing rather than false-alarm. The engine has a separate
`stale` primitive to detect the death itself.

### For your build

Your signals come from your boat survey (Step 1) and your `io_master.yml` (Step 3).
Every Modbus DI, relay, and analog channel you define in io_master becomes an `io.*`
signal. Every SignalK path on your N2K bus becomes an `sk.*` signal.

---

## 2. Primitives — what a condition can say

### `is` — digital state

```yaml
if: {io.high_water_salon: {is: true}}
if: {io.engine_running: {is: false}}
```

Exact match on a boolean or value. Use for dry-contact sensors (float switches, reed
switches) where the signal is on or off.

### `above` / `below` + `clear_at` — threshold with hysteresis

```yaml
if: {cerbo.house_soc: {below: 20, clear_at: 25}}
if: {sk.environment.water.temperature: {above: 95, clear_at: 90}}
```

Fires when the value crosses the threshold. Clears at `clear_at`, not at the threshold
itself. The gap between threshold and clear_at is the hysteresis band — it prevents
the rule from flapping on/off when the value hovers near the threshold.

**Always set `clear_at`.** Without it, a value that bounces around the threshold will
fire and clear every tick.

### `for` — debounce / sustain

```yaml
if: {io.bilge_pump_running: {is: true, for: 300s}}
when: {sk.yanmar_rpm: {above: 500, for: 30s}}
```

The condition must hold continuously for this long before it is considered met. Resets
if the condition goes false during the window. Use for:

- **Debounce:** ignore transient sensor spikes (`for: 3s` on a float switch in waves)
- **Sustained alarm:** pump running too long (`for: 300s` = 5 minutes)
- **Startup delay:** engine must be running 30 s before the flow alarm arms

Units: bare number = seconds. Suffix `s`, `m`, `h` for clarity.

### `rise` / `fall` — rate of change

```yaml
if: {sk.environment.outside.pressure: {fall: {by: 3, within: 1h}}}
if: {sk.exhaust.temperature: {rise: {by: 15, within: 5m}}}
```

Fires when the value changes by at least `by` within the `within` window. `rise` =
increasing, `fall` = decreasing. Use for trend detection: barometric drop, exhaust
temp climbing, fuel level falling.

### `starts` — cycling detection

```yaml
if: {io.bilge_pump_running: {starts: {at_least: 6, within: 60s}}}
```

The signal went off → on at least N times in the window. Detects pump short-cycling
(a sign of a stuck float switch or a pump losing prime). Cannot be combined with `for`.

### `stale` — data stopped arriving

```yaml
if: {sk.propulsion.main.revolutions: {stale: 60s}}
if: {cerbo.house_soc: {stale: 300s}}
```

No reading received for this long. Fires from the moment the silence exceeds the
specified duration, even if the signal's own `max_age` is shorter. A signal never
heard since engine start counts from start. Use to detect dead sensors, broken buses,
crashed adapters.

### `latch` — remember that something happened

```yaml
if:
  latch:
    set: {sk.yanmar_rpm: {above: 400, for: 5s}}
    reset: {sk.yanmar_rpm: {below: 100, for: 30s}}
```

True from the tick `set` is met until a tick `reset` is met. If both are true in the
same tick, `set` wins. In memory only — a restart forgets it. Use for state tracking:
"the engine has been running" (set on RPM up, reset on RPM down + 30 s).

### `all` / `any` / `not` — logic combinators

```yaml
if:
  any:
    - {io.high_water_salon: {is: true}}
    - {io.high_water_bow: {is: true}}

if:
  all:
    - {cerbo.house_soc: {below: 15}}
    - not: {io.shore_power_connected: {is: true}}
```

Combine conditions. `all` = AND (every sub-condition must be true). `any` = OR (at
least one). `not` = invert. Nest freely.

### `when` — arming gate

```yaml
when: {mode: {is: ANCHOR}}
when: {sk.yanmar_rpm: {above: 500, for: 30s}}
when:
  all:
    - {mode: {is: MOTOR}}
    - {sk.yanmar_rpm: {above: 400}}
```

The rule is inert unless `when` is true. Use for mode-dependent rules (anchor alarm
only at anchor, engine rules only while running). `when` accepts any primitive.

---

## 3. Actions — what a rule can do

### `alarm` — raise an alarm

```yaml
do:
  alarm: {severity: EMERGENCY}
```

| Severity | What happens |
|---|---|
| `EMERGENCY` | Horns + phone push + ship's log. Full response |
| `ALARM` | Phone push + ship's log |
| `NOTICE` | Phone push (quiet) |
| `LOG` | Ship's log entry only |

### `message` — notification text

```yaml
do:
  message: "EMERGENCY raw-water flow lost — check strainer / impeller"
```

Plain text with optional placeholders:

| Placeholder | Expands to |
|---|---|
| `{signal_name}` | Current value of that signal |
| `{signal_name:.0f}` | Formatted value (Python number format) |
| `{eta:signal:target}` | Time for signal to reach target at current rate |

ETA requires a `rise` or `fall` on the same signal in the rule. Output: `~12 min`,
`~1.5 h`, `under a minute`, `now`, or `? min` (no rate yet).

#### Conditional messages (`choose`)

```yaml
do:
  message:
    choose:
      - if: {io.bilge_pump_running: {is: true}}
        text: "High water — pump running, being overwhelmed"
      - text: "High water — pump NOT running, check pump / breaker"
```

The first matching `if` wins. The last entry (no `if`) is the default. Conditions
inside `choose` are evaluated every tick, so `for` and `starts` in them have history
when the alarm fires.

### `set` — drive a relay

```yaml
do:
  set: {io.yanmar_stop_flow_smoke: on}
```

Turns a relay on or off. **Guardrails:**

- Only relays whose `io_master` class is `ASSIST` or `PROTECT`
- `PROTECT` relays require `clear: manual` on the rule
- Two rules wanting different states on one relay: higher rank wins
- Rank: lockout rules > PROTECT relays > ASSIST relays

### `after_unacked` — ack-or-kill pattern

```yaml
do:
  alarm: {severity: EMERGENCY}
  after_unacked: {in: 15s, do: {set: {io.yanmar_stop_flow_smoke: on}}}
```

If the alarm is not acknowledged within `in` seconds, execute the `do` action. The
human has a window to override. This is how the reference build implements "alarm
first, kill second" — the engine stops only if nobody acknowledges within 15 seconds.

### `inhibits` — lockout precedence

```yaml
do:
  inhibits: [WF-201, WF-202, io.batteryprotect_ok_to_discharge]
```

While this rule is active, the listed rules and I/O functions are blocked. Use for
priority overrides: "during a flood, don't shed battery loads" (the pumps need power).

### `event` — ship's log entry

Every alarm writes to the ship's log by default. `event` adds a log entry for rules
that don't alarm.

### `clear` — how the alarm resets

| Value | Behavior |
|---|---|
| `auto` (default) | Clears when the trigger condition clears |
| `on_ack` | Clears when acknowledged |
| `manual` | Stays active until explicitly reset by the crew |

`PROTECT` actions (engine kills, charge disconnects) require `clear: manual`.

### `record` — where events go

| Value | Destination |
|---|---|
| `handover` | Ship's log AND watch-handover report |
| `log` | Ship's log only |
| `audit` | Engine audit log only (internal) |

Default: EMERGENCY/ALARM → handover, NOTICE → audit, output-only → log.

---

## 4. The lifecycle: dry_run → live

Every rule starts in `dry_run`. A dry_run rule evaluates normally but doesn't act —
it logs "would set io.X on" instead of setting it.

### Promotion sequence

1. **Write the rule.** Your AI session drafts it from your survey and signal list.
2. **Deploy as `dry_run`.** Reload the engine. The rule evaluates on real data.
3. **Watch it.** Check the engine log for "would fire" events. Verify they match your
   expectations. Run for days, not minutes — wait for the real conditions to occur.
4. **Simulate.** Use `simulate io.signal=value for 60s` to force the condition. Verify
   the rule fires, the message is correct, and no unexpected side effects occur.
   Simulated input is tagged `SIMULATED` on every event it causes.
5. **Promote to `live`.** Change `mode: live` and reload. One rule at a time.

### Simulation guardrails

- A simulated signal can never drive a `PROTECT` relay live — the engine forces dry_run
- A test session records relay states at start and restores them at end
- Test-activated lockouts block only other test actions, never real protection
- Test sessions auto-expire (default 1 min, max 1 h)

---

## 5. The degradation model

What still works when something fails.

| Failure | What happens | What doesn't work |
|---|---|---|
| **Pi dies** | Alarm panel (Tier 3) still shows sensor alarms via hardwired LED circuits. No phone notifications, no rules evaluation, no history logging | Automated relay actions (engine kill, load shedding) stop — the relays return to their de-energized (fail-safe) state |
| **One Modbus module dies** | The engine detects it (`stale` on that module's signals). Other modules continue. The alarm panel's hardwired channels for that module's sensors still work | Rules that read or write the dead module's signals stop evaluating. The module's relays drop to fail-safe |
| **One sensor dies** | The signal reads `UNKNOWN`. Rules depending on it stop firing (safe default). A `stale` rule on that signal fires a "sensor dead" alert | The specific alarm that sensor provides is blind. The panel's hardwired channel for that sensor still works |
| **Network dies** | Rules run on local data (Modbus, SignalK on the Pi). No phone notifications until network returns | Call-home ladder (phone push) is dead. The alarm panel and horns still work locally |
| **Ethernet gateway dies** | If the Pi has the RS485 CAN HAT, swap to it (it's the cold spare). Rules keep running after a brief interruption | During the swap, Modbus polling stops. The alarm panel is unaffected |
| **SignalK crashes** | `sk.*` signals go `UNKNOWN` → `stale` rules fire. `io.*` and `cerbo.*` signals still work | Any rule that depends on an `sk.*` signal stops evaluating until SignalK restarts |

### Design principle

The automation is advisory. Its failure is announced on the alarm panel (heartbeat LED
goes dark). The alarm panel is primary and standalone — it works with no Pi, no network,
no software. Manual override stays in copper. Every automated output has a physical
switch that the crew can flip to override the automation.

---

## 6. Writing rules for YOUR system

Your AI session uses this reference to write rules. The process:

1. **Start from your survey** (Step 1) — what do you want to monitor?
2. **Find the signal** — look up the sensor in your `io_master.yml` (the `io.*` name)
   or your SignalK data (the `sk.*` path).
3. **Pick the primitive** — `is` for on/off, `above`/`below` for thresholds, `stale`
   for dead sensors, `rise`/`fall` for trends.
4. **Add debounce** — almost every sensor needs `for:` to filter transients.
5. **Add a mode gate** — should this rule fire at dock? At anchor? Only while motoring?
6. **Write the message** — what does the crew need to know and do? Name the equipment,
   the location, and the immediate action.
7. **Set the severity** — EMERGENCY for danger to vessel/crew, ALARM for attention
   needed, NOTICE for information, LOG for history.
8. **Deploy `dry_run`** — watch it, simulate it, promote it.

### Rule of thumb

Start with monitoring rules (no `set` action). Get comfortable with the signal
namespace and the dry_run → live cycle before writing rules that move relays. A bad
monitoring rule sends a false notification. A bad relay rule stops your engine.

---

## Reference

- `automation-engine/spec/04-Interfaces/rules-schema.md` — the canonical format spec
- `automation-engine/spec/08-Workflows/` — the full catalog of worked examples (178 rules)
- `automation-engine/stacks/rules-engine/data/rules.yml` — the reference build's live rules
- `automation-engine/spec/04-Interfaces/io_master.yml` — every I/O point with its signal name
