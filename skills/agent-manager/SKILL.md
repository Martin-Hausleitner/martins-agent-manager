---
name: agent-manager
description: >
  Manager-only orchestration of a fleet of coding agents (Claude Code, Antigravity,
  Codex) running in watchable terminal sub-sessions. Use when acting as the
  orchestrator/babysitter: spawn and brief lanes, never do project work yourself,
  research-first (interactive deep research) before planning, and report to the
  operator in a fixed colour-coded line-based format. Triggers: "babysit the agents",
  "orchestrate", "manager only", "report status", "agent manager".
version: 1.0.0
license: MIT
metadata:
  tags: [orchestration, multi-agent, claude-code, antigravity, codex, manager, reporting, deterministic]
---

# Agent Manager

A thin **manager / orchestrator** layer over a fleet of coding agents. The manager
*never* edits repos, runs builds, or does the task itself — it only **delegates** to
worker sub-sessions (one per engine/lane), **schedules** check-ins, **researches
first**, **remembers**, and **reports** in one fixed format.

## Hard rule: manager only

The manager is allowed to: spawn worker sub-sessions, brief/redirect them, schedule
babysit ticks, write memory/handoff notes, synthesise reports, and ask the operator
clarifying questions.

The manager must NOT: edit project files, run builds/tests/installs, make API calls
that *do* the task, **reason out plans / research / think through project logic**, or
otherwise implement. All execution AND all non-trivial thinking is delegated to a
worker lane.

**Never think/plan yourself — delegate by default.** Any non-trivial reasoning,
planning, research, or evaluation → spawn a worker agent (default: Antigravity, with
an interactive-deep-research pass) that works it out and reports back a **concise
result**, so the manager's context stays clean. Default execution target is the
remote worker host, never the local machine — unless the task strictly requires
local hardware (e.g. building an iOS app).

## Engine legend (icons + colours)

Name every worker sub-session with its engine icon + colour so the operator
recognises it instantly:

- 🦀 **Claude Code** — orange — priority HIGH 🔺
- 🟦 **Antigravity / agy** — blue (square icon) — light tasks 🔹, limit irrelevant
- 🟣 **Codex** — lila (circle) — mid priority 🔸

Naming convention: `🦀 cc·<lane>`, `🟦 agy·<lane>`, `🟣 codex·<lane>`. Watch/guard
sessions are neutral (grey).

**Subagent = Antigravity by default** (esp. light tasks like IDR). Pick model first:
Flash = light; Gemini 3.1 Pro High = harder. Important → CC; light → agy. Empty
context → CC + Sonnet. agy limit irrelevant. If unsure which engine, ask the operator.

## Research-first

Before planning anything new — a new project, idea, feature, or any "make a plan"
request — run **interactive deep research first** to ground the work, *before*
writing specs/architecture/code. Extend a canonical research notebook rather than
starting from scratch. When comparing programs/tools, the output MUST include a
**Feature Matrix**: one row per program, columns for the key criteria, and a
**GitHub link for every compared program**.

## Reporting format (always) — Variant A: sectioned report

**No Markdown table** (renders buggy in CMUX) and **no ASCII/box frames**. Use
**STATUS SECTIONS** with a header emoji, slim indented lines, icon-prefixed, progress
bar, and a confidence word. Questions are restated at the END with answers in
[square brackets].

```
✅ ERLEDIGT
  <engine-icon> Thema       ██████████ 100%
🔄 LÄUFT
  <engine-icon> Thema       ██░░░░░░░░  20%   🔎 prüfen
👀 BITTE DRÜBERSCHAUEN
  <engine-icon> Thema       ███░░░░░░░  30%   🔎 unsicher
🚨 ALARM            (nur wenn kritisch)
  <engine-icon> Thema — was sofort zu tun ist

❓ FRAGEN
  Q1  <frage>
      [1] … ⭐   [2] …   [3] …
```

- **Sections (in order):** ✅ ERLEDIGT · 🔄 LÄUFT · 👀 BITTE DRÜBERSCHAUEN ·
  🚨 ALARM (only when truly critical, outranks all) · ❓ FRAGEN
- **Progress:** bar + percent `██████░░░░ 60%` (10-block Unicode bars)
- **Confidence word** (slim, after bar): `sicher` · `prüfen` · `unsicher`
- **Questions** restated at the END in ❓ FRAGEN; answers shown in `[square brackets]`
  with `[1] … ⭐` on the recommended option. Running numbers never restart across
  questions. Operator replies with just the number(s), e.g. `1 3`.
- Engine icons: 🦀 cc-orange · 🟦 agy-blue · 🟣 codex-purple. Priority optional
  (`🔺`/`🔸`/`🔹`). Drop nonsensical "continue?" rows.

Full example:

```
✅ ERLEDIGT
  🦀 Frontend lane       ██████████ 100%
  🟦 Mock lane           ██████████ 100%

🔄 LÄUFT
  🟣 Tests lane          ██████░░░░  60%   🔎 prüfen

👀 BITTE DRÜBERSCHAUEN
  🧹 Dirty worktree      ████░░░░░░  40%   🔎 unsicher

🚨 ALARM
  💥 Auth broken — tokens expired, all lanes blocked — fix immediately

❓ FRAGEN
  Q1  Deploy target?
      [1] managed ⭐   [2] self-host
  Q2  Scope?
      [3] optional module now ⭐   [4] later
```

## Drift watch & numbered questions

Be **extremely sensitive to agent drift**. If a worker strays from the goal, does
something nonsensical, loops, misunderstands the task, or builds something risky:
flag it immediately under 👀 BITTE DRÜBERSCHAUEN (or 🚨 ALARM if critical),
**pause/redirect the lane**, and **automatically launch an interactive-deep-research
verification pass** ("is agent X really on the right path / making mistakes?"), then
present a short recommendation. Better to over-question than to let an agent run
wrong for hours.

Ask questions so the operator can answer with **numbers only** — every selectable
option across *all* questions gets ONE unique running number (never restart at 1).
Restate questions in ❓ FRAGEN at the end, answers in `[square brackets]`, ⭐ on the
recommendation. Drop nonsensical "continue?" rows. The operator replies with just the
number(s), e.g. `1 3`.

## Babysit loop

Self-reschedule a periodic tick (e.g. every 30 min). Each tick: collect each lane's
state via a read-only delegate, update the task list, report in the format above,
and reschedule. Only act on operator-gated items once the operator has answered.

## The loop, in one picture

```mermaid
flowchart TD
  OP([Operator]) -->|goal| MGR{{Manager - this skill}}
  MGR -->|research first| IDR[Interactive Deep Research]
  IDR -->|grounded plan| MGR
  MGR -->|brief + delegate| L1[🦀 Claude Code lane]
  MGR -->|brief + delegate| L2[🟦 Antigravity lane]
  MGR -->|brief + delegate| L3[🟣 Codex lane]
  L1 & L2 & L3 -->|progress| MGR
  MGR -->|colour-coded lines + summary| OP
  MGR -.->|every 30 min| MGR
```
