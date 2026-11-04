<!--
Purpose: Entry point for the New Owner's Guide — what this repo is, who it is for, how to use it.
Version: v1 (2026-11-04)
Scope: PUBLIC — this is the document a new boat owner hands to their AI session.
Architecture: wiki/Home.md is the page index; this README is the elevator pitch.
Invocation: n/a (reference doc)
-->

# Ship's Automation — New Owner's Guide

An AI-readable engineering guide for building a monitoring and automation system on a
cruising sailboat. This is not a human install manual — it is a structured specification
that a new owner hands to their AI assistant (Claude, ChatGPT, or equivalent) and says
**"help me build this on my boat."**

## What this is

Eight open-source repositories document a complete ship's automation system built aboard
SV Unladen Swallow 2 (Hylas 51). That system includes:

- **Monitoring and rules engine** — SignalK hub, data-driven alarm/automation rules,
  history, dashboards (automation-engine)
- **Maintenance and inventory** — scheduling, spares, food, ship's log, cockpit display
  (vessel-ops)
- **Passage planning** — weather-informed routing, briefings, float plans (passage-planner)
- **Anchor alarm** — standalone GPS circle-fit anchor watch with touchscreen (anchor-alarm)
- **Alarm panel** — custom PCB with LED indicators, backlit labels, mute/test (alarm-panel)
- **Physical wiring** — Modbus I/O, relay pool, DIN rail, commissioning (wiring-io)
- **Entertainment** — shuffle music player with Bluetooth auto-play (music-player)
- **Manuals RAG** — local AI search over manufacturer PDF manuals (manuals-rag)

Those repos document **one boat's specific build**. This guide is the **adaptation layer**
that helps you build YOUR version on YOUR boat.

## How to use this guide

1. **Start with the [Decision Tree](wiki/00-Decision-Tree.md)** — figure out what you want.
2. **Fill out the [Boat Survey](wiki/01-Boat-Survey-Template.md)** — collect what you have.
3. **Pick an [Architecture Tier](wiki/02-Architecture-Tiers.md)** — match your requirements
   to a reference design.
4. **Run a [Gap Analysis](wiki/03-Gap-Analysis-Template.md)** — compare what you have to
   what you need.
5. **Build the [BOM](wiki/04-Reference-BOM.md)** — shopping list with alternatives.
6. **Read [Before You Build](wiki/05-Before-You-Build.md)** — marine electrical fundamentals.
7. **Follow the [Rules Tutorial](wiki/06-Rules-Authoring-Tutorial.md)** — write automation
   rules for your setup.
8. **Commission with the [Test Guide](wiki/07-Commissioning-Guide.md)** — verify it works.

The [Generic vs Lee-Specific](wiki/Generic-vs-Lee-Specific.md) guide explains which parts
of the reference repos are transferable patterns vs. one boat's configuration.

## The AI's role

Your AI assistant reads these documents and the reference repos, then helps you:
- Ask the right questions about your boat and your goals
- Design a system that fits your budget and technical ability
- Generate a BOM with specific parts and suppliers
- Write automation rules tailored to your equipment
- Walk you through the build, step by step
- Test and commission the finished system

The build is iterative. The [D/F Register](wiki/03-Gap-Analysis-Template.md#the-df-register)
is the methodology: discover → decide → implement → test → adjust. Your initial plan will
evolve, and that's expected.

## Reference repos

| Repo | What it covers |
|---|---|
| [automation-engine](https://github.com/shipboard-sa/automation-engine) | Rules engine, specs, I/O master table, workflows |
| [vessel-ops](https://github.com/shipboard-sa/vessel-ops) | Maintenance/inventory app (FastAPI + React + SQLite) |
| [passage-planner](https://github.com/shipboard-sa/passage-planner) | Passage planning pipeline (CLI + web) |
| [anchor-alarm](https://github.com/shipboard-sa/anchor-alarm) | ESP32 anchor alarm firmware |
| [alarm-panel](https://github.com/shipboard-sa/alarm-panel) | Alarm panel PCB (EasyEDA, JLCPCB) |
| [wiring-io](https://github.com/shipboard-sa/wiring-io) | Physical wiring, commissioning, Modbus I/O |
| [music-player](https://github.com/shipboard-sa/music-player) | Android shuffle player for boat stereo |
| [manuals-rag](https://github.com/shipboard-sa/manuals-rag) | Local RAG search over manufacturer PDF manuals |
