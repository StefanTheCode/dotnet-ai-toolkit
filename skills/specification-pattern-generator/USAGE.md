# How to use — Specification Pattern Generator

**What it is.** Implements the Specification pattern for EF Core — reusable, testable query criteria objects (filter/include/order/page) plus an evaluator — so you stop duplicating LINQ across repositories.

**When to reach for it.** The same Where/Include/OrderBy logic is repeated across the codebase, or you want testable query criteria.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Add the specification pattern to this project."
- "Dedupe these repeated EF queries into specs."
- "Make my query criteria reusable and unit-testable."

**Get the best out of it.** Let it scan for repeated query logic so the specs it creates target real duplication. Keep `AsNoTracking` default for reads.

**Won't do.** It won't wrap one-off queries in ceremony — it'll tell you when a plain query is simpler.
