# HEARTBEAT-SPEC.md — Live Status Dashboard

> Spec for a terminal-UI live dashboard that the manager's heartbeat pane displays.
> The TUI library is **TBD by a pending IDR** (`agy-idr-termui`).
> All rendering must work on both macOS and CMUX (all-black dark bg, emoji icons).

---

## Design principles

- **Calm, line-based, emoji-keyed** — same style as the report format.
- **No ASCII box frames.** No table borders.
- **Bold for every key number** and every lane name.
- **Emoji icons as visual anchors** — they must render correctly on the CMUX iOS app.
- **ANSI colours** for the TUI variant (chat variant falls back to emoji + Markdown bold).
- Sections separated by a slim `───` divider line.
- Refresh: live poll (suggested interval 10 s); configurable.

---

## Section 1 — System

```
───
💻 SYSTEM
  RAM    **12.4 GB** / 16.0 GB   ████████░░  77%
  CPU    **34%** load (1 min avg)
  Disk   /           **42 GB** free / 500 GB
  Disk   /data       **218 GB** free / 1 TB
```

Fields:
- RAM used / total, Unicode bar, percent.
- CPU 1-min load average, percent.
- One line per relevant mount: path, free, total.

---

## Section 2 — Docker containers

```
───
🐳 DOCKER
  **container-name**    🟢 up 3d   CPU **1.2%**   MEM **234 MB**
  **another-service**   🟢 up 1h   CPU **0.4%**   MEM **88 MB**
  **idle-worker**       🔴 exited
```

Fields per container:
- Name (bold), status emoji (🟢 running / 🔴 stopped / 🟡 restarting), uptime, CPU%, MEM (MB).
- Only running containers show CPU/MEM; stopped containers show status only.

---

## Section 3 — Provider token budgets

```
───
🪙 TOKEN BUDGETS
  🦀 Claude Code    weekly **4 200** / 10 000   ████░░░░░░  42%
                    daily  **840** / 2 000       ████░░░░░░  42%
                    session **120** / 500        ██░░░░░░░░  24%
  🟣 Codex          weekly **6 100** / 10 000   ██████░░░░  61%
                    daily  **1 050** / 2 000     █████░░░░░  53%
  🟦 Antigravity    (limit irrelevant — no tracking)
```

Fields:
- Provider icon + name, budget type (weekly / daily / session), used / total, bar, percent.
- When usage exceeds 80%: bar colour → amber (ANSI yellow); label → `⚠️ low`.
- When usage exceeds 95%: bar colour → red (ANSI red); label → `🚨 critical`.
- Antigravity shown as "limit irrelevant" — no bar needed.

---

## Section 4 — Per-lane research history

```
───
🔬 RESEARCH ACTIVITY
  🦀 **frontend-lane**
      🧠 IDR  🔁3   🕐 2026-06-09 14:22   last: "NextJS routing strategy"
      🔬 DR   🔁1   🕐 2026-06-08 09:00
  🟣 **tests-lane**
      🧠 IDR  🔁1   🕐 2026-06-09 11:05   ⚡ running NOW
  🟦 **mock-lane**
      (no research runs)

  📜 HISTORY (last 10 runs, newest first)
      🕐 14:22  🦀 frontend-lane    🧠 IDR  "NextJS routing strategy"
      🕐 11:05  🟣 tests-lane       🧠 IDR  running… ⚡
      🕐 09:00  🦀 frontend-lane    🔬 DR   "Component library comparison"
```

Icons:
- `🔬` = Deep Research (general)
- `🧠` = IDR (interactive deep research)
- `🔁 N` = how many times this type has run for this lane this session
- `🕐 <timestamp>` = last run start time (local time, `YYYY-MM-DD HH:MM`)
- `⚡` = actively running right now

---

## Section 5 — Per-lane status and 2-day history

```
───
📊 LANES
  🦀 **frontend-lane**   🔺 HIGH    ██████████ 100%  ✅ done
  🟣 **tests-lane**      🔸 MID     ██████░░░░  60%  🔄 running
      🧠 IDR ⚡ running
  🟦 **mock-lane**       🔹 LOW     ██████████ 100%  ✅ done

  📅 2-DAY SUMMARY
      2026-06-08  frontend-lane ✅  mock-lane ✅  tests-lane 🔄 40%→60%
      2026-06-09  tests-lane 🔄 60% (today)
```

Fields per lane:
- Engine icon, lane name (bold), priority marker (🔺/🔸/🔹), progress bar + percent, status emoji.
- Research indicator line if research is running or ran recently (same icons as Section 4).
- 2-day history: one line per day, one token per lane showing its final state that day.

---

## Section 6 — Manager heartbeat

```
───
💓 MANAGER
  Last tick   🕐 2026-06-09 14:28   **1 min ago**   🟢 LIVE
  Next tick   🕐 2026-06-09 14:58   in **29 min**
  Host        worker-host-A
  Engine      🦀 Claude Code
```

- Last tick time + how long ago (bold if >35 min → 🔴 STALE).
- Next scheduled tick countdown.
- Host and engine (anonymised host label — no IPs or internal hostnames in public builds).

---

## Rendering notes

- **Terminal-UI library**: TBD. Must support: live refresh, ANSI 256/true-colour,
  Unicode + emoji rendering, all-dark background. Candidates to be evaluated by
  `agy-idr-termui` IDR before implementation.
- Target environments: macOS terminal (dark theme) AND CMUX iOS app (all-black bg).
  Emoji icons must be tested on both before the library is chosen.
- The dashboard is read-only; no interactive input required.
- Suggested update strategy: full re-render every 10 s (configurable via env var
  `HEARTBEAT_INTERVAL_SECS`).
- All timestamps in local time; include timezone suffix when ambiguous.

---

## Data sources (abstract — implementation-specific)

| Section | Data origin |
|---|---|
| System | `/proc/meminfo`, `/proc/loadavg`, `df` (Linux) or `vm_stat`, `sysctl`, `df` (macOS) |
| Docker | Docker Engine API (`GET /containers/json`, stats stream) |
| Token budgets | CodexBar-compatible data file or API; path configurable |
| Research history | Manager state file (JSON), updated by IDR/DR runners on each run |
| Lane status | Manager state file (JSON), updated each babysit tick |
| Heartbeat | Manager last-tick file (plain timestamp), updated each babysit tick |

---

*This spec is implementation-neutral. The build lane should start with an IDR to
choose the TUI library, then implement section by section, testing on both target
environments before merging.*
