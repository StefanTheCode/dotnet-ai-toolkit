# How to use — Architecture Tests Generator (NetArchTest)

**What it is.** Generates architecture tests (NetArchTest/ArchUnitNET) that fail the build on rule violations — layer dependencies, naming, sealed/internal conventions, module boundaries — tailored to your structure.

**When to reach for it.** Stopping architectural drift, enforcing the dependency rule, or guarding modular-monolith boundaries.

**How to use it.** Point it at your solution (path or GitHub URL). Example prompts:
- "Add architecture tests to enforce my Clean layers."
- "Stop controllers from referencing DbContext."
- "Guardrail tests so my modules can't reference each other."

**Get the best out of it.** Let it detect your architecture so rules fit (Clean vs Vertical Slice vs modules). Keep a handful of high-value rules. Pairs with `clean-architecture-scaffolder`, `modular-monolith-generator`, and the architecture-reviewer agent.

**Won't do.** It won't impose dogmatic rules that don't match your chosen style.
