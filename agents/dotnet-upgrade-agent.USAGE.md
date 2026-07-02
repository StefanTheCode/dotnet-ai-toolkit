# How to use — .NET Upgrade Agent

**What it is.** An agent that plans and (with approval) executes a .NET version upgrade across a solution — TFM bumps, breaking-change remediation, obsolete-API replacement, package updates, and per-stage verification.

**When to reach for it.** Upgrading to a newer .NET (e.g. 6→8→9→10) or migrating target frameworks.

**How to use it.** Point it at your solution (path or GitHub URL). Example prompts:
- "Upgrade this solution from .NET 6 to .NET 9."
- "Plan a migration to .NET 10 and flag the breaking changes."
- "What obsolete APIs will break when I upgrade?"

**Get the best out of it.** Let it upgrade one major at a time and verify (build + tests) between steps. Review its plan before it edits — it asks first. It defers deep rewrites to `legacy-modernization-assistant`.

**Won't do.** It won't power through a red build or skip verification to finish faster.
