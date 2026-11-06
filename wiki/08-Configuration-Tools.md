<!--
Purpose: How to use the browser-based configuration editors and audit viewer.
Version: v1 (2026-11-05)
Scope: PUBLIC — describes the tools that ship with the automation-engine repo.
Architecture: References automation-engine/tools/ for each tool's own README.
Invocation: n/a (reference doc)
-->

# Configuration Tools

The automation engine ships with browser-based editors for every data file the
engine reads. No install, no build step — serve the repo with any HTTP server
and open the tool in a browser.

These tools are how you inspect, edit, and understand the data that drives the
engine's rules, signals, I/O map, and decision history.

---

## Quick start

From the automation-engine repo root:

```bash
python -m http.server 8090
```

Then open any tool at `http://localhost:8090/tools/<name>/`.

---

## The tools

### Rules editor (`tools/rules-editor/`)

View and edit `rules.yml` — the alarm and automation rules the engine evaluates
every cycle.

- Every rule rendered as a human-readable sentence ("when yanmar_rpm > 3200
  for 5s, set alarm HIGH")
- Inline threshold tuning — adjust trigger values without editing raw YAML
- New rule builder with signal pick lists populated from `signals.yml`
- Step-by-step wizard for common patterns (high/low threshold, state change,
  rate-of-change alarm)
- Cross-references `signals.yml`, `engine.yml`, and `io_master.yml` for
  validation

Use this during [Step 6 — Rules Authoring](06-Rules-Authoring-Tutorial.md) to
build rules interactively instead of hand-editing YAML.

### Signals editor (`tools/signals-editor/`)

View and edit `signals.yml` — the signal catalog that tells the engine where
every data point comes from and how to interpret it.

- Signals grouped by prefix (`sk.*` from SignalK, `io.*` from I/O modules,
  `cerbo.*` from Victron, etc.)
- Type inference badges (bool, real, string, position)
- Optional live polling — connect to a running engine to see current values
  alongside definitions
- New signal builder with prefix-specific defaults

Use this when adding new sensors or data sources to your system.

### I/O master editor (`tools/io-editor/`)

View and edit `io_master.yml` — the relay, digital input, analog input, and
panel master table.

- Every channel shown with its current state (unverified → verified → live)
- Module grouping (which channels belong to which Waveshare module)
- Cross-reference with `signals.yml` for signal-to-channel mapping

Use this during [Step 7 — Commissioning](07-Commissioning-Guide.md) to track
which channels have been proven and which are still waiting.

### Audit viewer (`tools/audit-viewer/`)

Read-only viewer for the engine's decision history — every rule evaluation,
the input values that triggered it, relay readback proof, and alarm delivery
records.

- Timeline view with filtering by rule, function, outcome, and time range
- Signal history graphs
- Relay proof table — readback confirmation that hardware followed software
- Works against PostgreSQL (production) or offline JSONL files

Use this after commissioning to verify the engine is making the decisions you
expect, and to troubleshoot unexpected alarms or relay activations.

### Deploy API (`tools/deploy-api/`)

The save pipeline that connects the editors to a live engine. When you click
Save in any editor:

1. Validates the YAML
2. Writes it to the engine's data directory
3. Runs `engine check` for cross-validation
4. Commits the change to the deployed git checkout
5. Hot-reloads the engine (for rules) or advises restart (for signals/I/O)

Optional — without the deploy API, use the Download button in any editor to
save YAML locally, then deploy manually.

---

## Where they fit in the workflow

| Guide step | Tool |
|---|---|
| [Step 6 — Rules Authoring](06-Rules-Authoring-Tutorial.md) | Rules editor, signals editor |
| [Step 7 — Commissioning](07-Commissioning-Guide.md) | I/O master editor, audit viewer |
| Ongoing operation | All four — tune thresholds, add signals, review decisions |

---

## Further reading

Each tool has its own README in `tools/<name>/README.md` in the
[automation-engine](https://github.com/shipboard-sa/automation-engine) repo
with full API docs, command-line options, and architecture notes.
