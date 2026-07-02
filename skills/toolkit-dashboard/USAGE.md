# How to use — Toolkit Dashboard

**What it is.** A "home" skill that generates a single self-contained `dashboard/index.html` — a localhost web page that catalogs every skill and agent in the toolkit and shows the history of skill/agent runs with their findings. Zero dependencies, one file, runs anywhere.

**When to reach for it.** Run it **first** when you open the toolkit, and again any time you add, update, or run a skill/agent so the overview stays current.

**How to use it — just ask Claude.** You don't run anything yourself. Say "show me the dashboard" / "open the toolkit" / "refresh the dashboard" and Claude runs the generator (Bash) and opens `dashboard/index.html` for you (presents it in Cowork, or serves it in Claude Code). Members get the same: they invoke the skill, Claude builds and opens it.

Example prompts that trigger it: "open the dashboard", "show me the toolkit", "refresh the dashboard", "what skills do I have", "I want to see the run results", "build the case-study showcase".

Manual fallback (only if you want a terminal): `python3 skills/toolkit-dashboard/scripts/build_dashboard.py` (auto-detects the toolkit root — no `--root` needed), then open `dashboard/index.html` or serve with `python3 -m http.server 8080`.

**How to get the best out of it.**
- **Record your runs.** After running any skill/agent, drop a JSON record in `runs/` (schema in `references/run-record-schema.md`) and re-run the generator — that's how results appear in the History tab. The richer your run records, the more useful the dashboard.
- **Let skills self-describe.** Add `category:` and `version:` to a skill's frontmatter and the dashboard picks them up automatically — no central list to maintain. `catalog-meta.json` is the fallback for items that don't declare them.
- **Keep it as the front door.** Point members here first; the cards link to each skill's `USAGE.md` so people know how to drive each tool.
- **Re-run freely.** It's idempotent — regenerating just overwrites `dashboard/index.html`.

**What it won't do.** It doesn't *execute* skills — it catalogs them and displays results you record. It serves on localhost (or `file://`); it's not a hosted web app. It reads only the toolkit folders, so anything you want shown (a run, a category) has to exist as a file.
