---
name: toolkit-dashboard
description: Build, refresh, and OPEN the TheCodeMan Toolkit dashboard — a self-contained web page that catalogs every skill and agent and shows past run results. Use this skill FIRST when opening the toolkit, or whenever the user wants an overview of skills/agents, a dashboard, a home page, run results/history, or says "show me the toolkit / open the dashboard / refresh the dashboard / run the dashboard". When triggered, Claude runs the generator itself with the Bash tool and then opens/presents the dashboard for the user — the user never has to run Python manually. Always use this skill for toolkit overview and dashboard requests.
category: Meta / Tooling
version: 1.0.0
---

# Toolkit Dashboard

Generate a single, self-contained `dashboard/index.html` that gives a clear visual overview of the whole toolkit: every skill and agent (name, category, version, description, docs), plus a history of skill/agent runs and their findings. It runs on localhost with one command and has zero runtime dependencies.

## When this runs

- **First thing** when someone opens the toolkit and wants to see what's inside.
- Whenever a skill or agent is **added, updated, or run** — regenerate so the dashboard stays current.
- When the user asks for an overview, a dashboard, run history, or "what do I have".

## What it does

1. **Scans `skills/*/SKILL.md`** — parses YAML frontmatter (`name`, `description`, optional `category`, `version`), and notes whether each skill has a `USAGE.md` and a `references/` folder.
2. **Scans `agents/*.md`** (excluding `*.USAGE.md`) — parses frontmatter (`name`, `description`, `tools`, optional `category`, `version`).
3. **Merges `catalog-meta.json`** (optional, at the toolkit root) — fills in `category`/`version`/`tags` for any item that doesn't declare them in frontmatter.
4. **Loads `runs/*.json`** — structured run records (see the schema below) and renders them as a searchable, filterable history with verdict badges and findings.
5. **Writes `dashboard/index.html`** — one file, all data embedded, no external requests. Works over `file://` or a local server.

## How Claude runs it (no terminal for the user)

When this skill triggers, **Claude builds and opens the dashboard for the user** — they should never have to run Python themselves.

1. **Build it** with the Bash tool. The generator auto-detects the toolkit root, so no arguments are needed:
   ```bash
   python3 "<toolkit>/skills/toolkit-dashboard/scripts/build_dashboard.py"
   ```
   (`--root <toolkit>` also works if you want to be explicit.)
2. **Open it for the user:**
   - **Cowork:** present `dashboard/index.html` with the file-presentation tool so they open it in one click.
   - **Claude Code / terminal:** give them the path and offer to serve it — `cd dashboard && python3 -m http.server 8080` → `http://localhost:8080`.
3. **(Optional) Also build the case-study showcase:**
   ```bash
   python3 "<toolkit>/skills/toolkit-dashboard/scripts/build_showcase.py" --target "Project Atlas"
   ```
   → `showcase/index.html` (a branded, screenshot-ready results page).

The page is a single self-contained file (data embedded, zero dependencies), so opening `dashboard/index.html` directly via `file://` always works too. Re-running is cheap and idempotent — just rebuild after any change.

## The `runs/` convention (how results show up)

For a skill or agent run to appear in the dashboard, drop a JSON file in `runs/` following the schema in `references/run-record-schema.md`. Minimal example:

```json
{
  "id": "2026-06-16-ef-core-atlas",
  "tool": "ef-core-query-optimizer",
  "type": "skill",
  "target": "Project Atlas",
  "date": "2026-06-16",
  "verdict": "issues",
  "summary": "In-memory filtering + missing AsNoTracking in Configurations/Queries/List.cs",
  "findings": [
    { "severity": "medium", "title": "No projection / over-fetch", "location": "Configurations/Queries/List.cs" },
    { "severity": "medium", "title": "Missing AsNoTracking", "location": "Configurations/Queries/List.cs" }
  ],
  "report": "TEST-RESULTS.md"
}
```

**Verdict values** (drive the badge color): `clean`, `pass`, `issues`, `critical`, `generated`.
**Severity values:** `low`, `medium`, `high`, `critical`.

When you run any other skill/agent and want it recorded, write a matching record to `runs/` and re-run the generator. The dashboard updates automatically.

## Output

- `dashboard/index.html` — the dashboard (overwrite each run).
- Console prints counts (skills, agents, categories, runs) and the launch command.

## Principles

- **One file, no dependencies.** Members can open it anywhere; it never phones home.
- **Self-describing.** Categories/versions come from the skills themselves (frontmatter) first, `catalog-meta.json` second — no hardcoded list to maintain.
- **Re-run cheap.** Regenerating is idempotent; run it after every change.
- **On brand.** Deep purple-navy background, yellow accents, dark cards — matches TheCodeMan visual identity.
