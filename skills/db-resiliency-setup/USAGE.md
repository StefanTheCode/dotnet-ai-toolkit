# How to use — Database Resiliency Setup

**What it is.** Adds EF Core resiliency — retry on transient failures, command timeouts, and the correct execution-strategy pattern for manual transactions (the caveat that breaks naive retry setups).

**When to reach for it.** Transient DB errors (timeouts, failovers, throttling), or after enabling retry and finding your transactions break.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Add connection retry to my DbContext."
- "My multi-step transaction breaks with retry enabled — fix it."
- "Make these DB writes resilient and idempotent."

**Get the best out of it.** Let it check for manual `BeginTransaction` usage — those must move inside the execution strategy. Ask about idempotency if your unit of work can be safely retried. Pairs with `dbcontext-config-auditor`.

**Won't do.** It won't retry logic errors (constraint violations) — only known transient faults.
