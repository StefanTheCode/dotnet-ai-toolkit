# How to use - Codebase Health Dashboard

**What it is.** The "run everything" tool. Point it at your .NET repo and it runs the toolkit's auditors, scores eight categories with a fixed rubric, and writes a single self-contained **health dashboard** - one overall score, per-category scores, the findings behind each, and the top fixes.

**When to reach for it.** A whole-repo health check, an overall code-quality score, a pre-release report card, or to track your codebase score week to week.

**How to use it.** Point it at your project/solution or a GitHub URL. Example prompts:
- "Run a health check on this repo."
- "Give my .NET solution an overall score and a report card."
- "How healthy is this codebase? Show me where I stand."

**Get the best out of it.**
- **Re-run it weekly** and watch the score climb - that is the point. Each run saves a record in `health/`, so you can track the trend.
- Act on the **Top actions** first - they are the fastest wins.
- The scoring is **deterministic** (critical -25, high -12, medium -5, low -2 per category, from 100), so the number is comparable run to run.
- Share your score (or a screenshot) in the community - it is great for the "show your results" thread.

**Won't do.** It won't invent findings to lower your score - it credits what is already healthy. Index/CVE claims that need a live DB or restore are marked "verify", same as the underlying tools.
