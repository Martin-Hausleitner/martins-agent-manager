# 🧩 Skills-Zentralisierung — Plan

**Stand:** 2026-06-27 · **Autor:** 🤖 Martins Agent · **Ziel:** EIN kanonischer Skill-Store statt ~10 Duplikate + Dutzende verstreute Stores.

---

## 🔍 Befund (Ist-Zustand) — das Chaos

**~10 Riesen-Mirrors à ~1.500 Skills** = derselbe Marketplace-Pack ~10-fach dupliziert (5× Mac + 5× vcvm):

| Root (je Mac **und** vcvm) | ~Skills | Was |
|---|---|---|
| `~/.claude/skills` | ~1540 | Marketplace-Pack |
| `~/.codex/skills` | ~1670 | Marketplace-Pack (Dup) |
| `~/.gemini/skills` | ~1505 | Marketplace-Pack (Dup) |
| `~/.agents/skills` | ~1515 | Marketplace-Pack (Dup) |
| `~/.openclaw/skills` | ~1475 | Marketplace-Pack (Dup) |

**+ kleine, KURATIERTE Stores** (hier liegen deine echten Skills verstreut):
`agent-mon/skills` · `.hermes/skills` (33) · `.hermes/hermes-agent/skills` (26) · `.openclaw/workspace/skills` (111) · `.config/opencode/skills` (42) · `agent-live-status/skills` · `martins-agent-manager/skills` · `deep-research-repo/skills` …

**+ Dutzende per-Projekt** `vk-repos/*/.claude/skills`, `*/.agents/skills` (rallly, orwin, credo, Fintaro, ReplyManager, onlyapi-chrome, …).

🔴 **Probleme:** massive Duplikation (Platte + Drift), deine ~30 eigenen Orchestrierungs-Skills mit dem Vendor-Pack vermischt, kein Single-Source, Updates laufen nie überall an.

---

## 🎯 Zielbild — `skills-hub` (Single Source of Truth)

Ein **Git-Repo** `skills-hub` (privat, GitHub) mit klarer Trennung:

```
skills-hub/
  authored/     # 👑 DEINE Skills (ask-human, openspec-first, fleet-overnight-report,
                #    agent-fleet-control, power-quality-gate, integrative-deep-research …)
                #    versioniert, EINZIGE Quelle
  vendored/     # 📦 Marketplace-Pack — EINE deduplizierte Kopie (statt 10×)
  MANIFEST.md   # Index: Skill · Kategorie · Herkunft(authored/vendored) · welche Engines
  sync/skills-sync.sh   # verteilt Hub → alle Engine-Roots (Mac + vcvm)
```

Jeder Engine-Root (`~/.claude/skills`, `~/.codex/skills`, …) wird **Symlink** (oder rsync-Mirror, falls eine Engine keine Symlinks mag) auf den Hub.

---

## 🛠️ Migrations-Schritte (Lane, OpenSpec-first)

1. **Inventar+Klassifizieren** (Skript): jede `SKILL.md` über ALLE Roots hashen → identische = Vendor-Pack, unique = authored/per-Projekt. Report `MANIFEST.md`.
2. **Hub anlegen:** `skills-hub` Repo (privat). `authored/` = deine ~30 (aus den kuratierten Stores eingesammelt + dedupliziert). `vendored/` = 1 Kopie des Packs.
3. **Dedupe:** die ~10 Mirrors → je Engine-Root ein **Symlink** auf `skills-hub` (vorher Backup). Verifizieren dass jede Engine danach noch lädt (sonst rsync-Mirror-Fallback).
4. **Per-Projekt:** echte projekt-eigene Skills bleiben im Repo; geteilte/authored → Symlink auf Hub.
5. **Sync:** `skills-sync.sh` via launchd (Mac) + cron (vcvm): `git pull` Hub → relink/rsync auf beiden Hosts. Neuer Skill → 1× in Hub committen → überall da.
6. **Safety-Gate:** Backups je Root, Engine-Load-Test nach Relink, Roll-back-Pfad.

**Akzeptanz:** 1 Single-Source · ~10 Mirrors → Symlinks · Dedup-Ratio belegt · Sync auf Mac+vcvm grün · jede Engine lädt Skills weiter · `MANIFEST.md` vollständig.

**Disk-/Sanity-Gewinn:** ~10×1.500 SKILL-Bäume → 1.

---

## ✅ Schon erledigt (heute)
- 🛠️ **Beeper/Signal-Skill gefunden:** `ask-human` → `lib/beeper-send.sh` (lokale Beeper-Desktop-API). Empfänger **David Krausi (Signal)** verifiziert.
- 📇 Voll-Inventar Mac + vcvm (oben).
