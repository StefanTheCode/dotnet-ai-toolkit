# How to use — Database Performance Auditor (agent)

**What it is.** An agent that audits a whole .NET data layer — EF query patterns, indexing, DbContext config, resiliency, bulk-op opportunities — and produces a ranked report with fixes.

**When to reach for it.** "Why is my data layer slow", a pre-release DB performance pass, or a second opinion on EF usage across a repo.

**How to use it.** Point it at a repo/solution (path or GitHub URL); it explores the data layer itself. Example prompts:
- "Audit this repo's database performance."
- "Why is my data access slow across the app?"
- "Pre-release EF performance review."

**Get the best out of it.** Give it the whole solution so it sees config + queries together. Act on the ranked order. For deep single-query rewrites use `ef-core-query-optimizer`; for indexes use `ef-index-advisor`.

**Won't do.** It can't read your live query plan — index/plan claims come marked "verify".
