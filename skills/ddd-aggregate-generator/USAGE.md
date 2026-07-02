# How to use — DDD Aggregate Generator

**What it is.** Generates DDD building blocks — aggregate roots that protect invariants, value objects, and domain events — instead of anemic entities with public setters.

**When to reach for it.** Modeling a domain, turning an anemic entity rich, extracting value objects, or adding domain events.

**How to use it.** Point it at your project (path or GitHub URL) and describe the rules. Example prompts:
- "Model an Order aggregate: needs ≥1 line, can be confirmed only when pending."
- "Turn this anemic Customer entity into a proper aggregate."
- "Extract Money and Email value objects."

**Get the best out of it.** State the invariants explicitly — they become guarded factory/method logic. Let it match your existing `Entity`/`AggregateRoot` base types. Pair with `result-pattern-scaffolder` for method return types.

**Won't do.** It won't over-engineer a simple CRUD table into a heavy aggregate — it'll tell you when a plain entity is enough.
