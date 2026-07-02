# How to use — Modular Monolith Generator

**What it is.** Sets up or tightens a modular monolith — modules with `internal`-by-default code, small public contracts, in-process messaging, and a guardrail test that stops modules reaching into each other.

**When to reach for it.** Structuring a new app, auditing module boundaries, or splitting a tangled monolith into modules.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Structure this as a modular monolith: Orders, Billing, Catalog."
- "Audit my module boundaries and tell me what leaks."
- "Add a NetArchTest so modules can't reference each other's internals."

**Get the best out of it.** Ask for the guardrail test — boundaries that aren't enforced erode. Keep the shared kernel tiny. Treat it as a future-proof path to services without paying the distributed tax now.

**Won't do.** It won't split into microservices — that's the upgrade/modernization agents' job.
