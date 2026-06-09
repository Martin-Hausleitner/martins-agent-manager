---
name: cmux-control
description: >
  Reliably drive the operator's LOCAL cmux (Mac) from the manager session: show a
  markdown file / open a website / open a new terminal in a dedicated right-side
  "📄 preview" workspace, on command. Use when the operator says "show me <file>",
  "open <url>", "give me a terminal", or wants something displayed on the right.
version: 1.1.0
metadata:
  tags: [cmux, local-control, preview, markdown, browser, terminal, mac, dock, custom-sidebars]
---

# cmux Control (local, Mac)

The cmux CLI controls the operator's local cmux app over a Unix socket. It is
**Mac-local** (an allowed exception to "everything on remote VM"). Binary:
`/Applications/cmux.app/Contents/Resources/bin/cmux` (alias on PATH: `cmux`).
Always set `CMUX_QUIET=1`.

## Concepts
- **Window** → **Workspace** (tab group) → **Pane** (split) → **Surface** (tab: terminal or browser).
- Refs: `workspace:N`, `pane:N`, `surface:N`. From a non-cmux shell there is NO caller
  context, so `cmux open` MUST be given an explicit `--pane` (otherwise: `not_found:
  Source surface not found`).

## TESTED reliable patterns (verified 2026-06-09)

### Show a markdown / file / image on the RIGHT (preview)
Markdown opens as a rendered preview tab; other files as file previews.
```bash
CM=/Applications/cmux.app/Contents/Resources/bin/cmux; export CMUX_QUIET=1
# 1) ensure a dedicated preview workspace exists
REF=$("$CM" workspace list | grep '📄 preview' | grep -oE 'workspace:[0-9]+' | head -1)
[ -z "$REF" ] && REF=$("$CM" workspace create --name "📄 preview" --focus false | grep -oE 'workspace:[0-9]+')
# 2) resolve its pane (REQUIRED when called from outside cmux)
PANE=$("$CM" list-panes --workspace "$REF" | grep -oE 'pane:[0-9]+' | head -1)
# 3) open — markdown renders, focus brings it to front
"$CM" open /path/to/file.md --workspace "$REF" --pane "$PANE" --focus true
```
Multiple files → multiple tabs in the same pane. For a remote VM file, first copy it local
(`scp remote-vm:/path .` ) or `ssh remote-vm cat > /tmp/x.md`, then open the local copy.

### Open a website / URL
Using `cmux browser open` is cleaner and avoids "Source surface not found" errors when called from a non-cmux shell:
```bash
# Open in the preview workspace
CM=/Applications/cmux.app/Contents/Resources/bin/cmux; export CMUX_QUIET=1
REF=$("$CM" workspace list | grep '📄 preview' | grep -oE 'workspace:[0-9]+' | head -1)
"$CM" browser open "https://example.com" --workspace "$REF" --focus true
```

### New terminal on the right
Use `cmux workspace create` (canonical command) or the legacy alias `new-workspace`:
```bash
# Create new workspace with a running command
cmux workspace create --name "🖥 <name>" --command "<cmd>" --focus true
```

### Switch to / Focus a Workspace
```bash
# Switch active workspace focus
cmux workspace select workspace:N
```

### Close Workspace / Surface
```bash
# Close a workspace
cmux workspace close workspace:N

# Close a specific surface (tab)
cmux close-surface --surface surface:N --workspace workspace:M
```

### Naming & Coloring convention (operator preference)
cmux workspace titles are **clean names — NO `agy·`/`cc·`/`exodus·`/`manager·`/`⚙ ` text prefix**. Keep the category/engine ICON only: `🟦 main-app`, `🟦 mock-server`, `🦀 <cclane>`, `📄 preview`, `🕒 heartbeat`, `⚙ health-arch`.
- **Rename**: `cmux workspace-action --action rename --workspace workspace:N --title "<icon> <name>"`
- **Color**: `cmux workspace-action --action set-color --workspace workspace:N --color <ColorName|#hex>`
  - *Named colors*: `Red`, `Crimson`, `Orange`, `Amber`, `Olive`, `Green`, `Teal`, `Aqua`, `Blue`, `Navy`, `Indigo`, `Purple`, `Magenta`, `Rose`, `Brown`, `Charcoal`.

## Dock & Custom Sidebars (Personalization)

### Dock Configuration
The Dock pins terminal controls to the right sidebar, letting you run persistent CLI interactive commands (e.g. dashboards, git, dev-servers).
- **Paths**: Project-specific `.cmux/dock.json` or global `~/.config/cmux/dock.json`.
- **Edit**: Edit the JSON file, then reload to apply:
```bash
cmux reload-config
```
- **Structure**:
```json
{
  "controls": [
    {
      "id": "my-logs",
      "title": "Logs",
      "command": "tail -f ./logs/development.log",
      "cwd": ".",
      "height": 200
    }
  ]
}
```

### Custom Sidebars (Beta)
Write runtime-interpreted SwiftUI-style files in `~/.config/cmux/sidebars/` (e.g. `mine.swift`) and right-click the sidebar button to select them. Runs dynamically (re-evaluates ~1/s) and supports actions like `cmux("workspace.select", workspace_id: w.id)`.

## Diagnostics & Inspection
- **Topology Check**: `cmux tree --all` (inspect all windows, workspaces, panes, and surfaces)
- **Identify Context**: `cmux identify --json` (shows current context: caller pane/surface/workspace)
- **Ping / Health**: `cmux system ping` (check if daemon socket is alive) / `cmux surface-health`
- **Visual Attention flash**: `cmux trigger-flash --surface surface:N` (pulses/flashes target tab to grab operator's attention)

## Default behaviour for the manager
When the operator asks to *see* something (a markdown report, a PR README, a URL, a
log), DON'T paste walls of text — **open it in the 📄 preview workspace** with the
snippet above and tell them it's on the right. Build/keep the preview workspace once;
reuse its pane.
