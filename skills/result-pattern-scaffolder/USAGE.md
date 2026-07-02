# How to use — Result Pattern Scaffolder

**What it is.** Adds a Result/Error type so expected failures are returned as values instead of thrown, plus a single place to map those errors to HTTP responses.

**When to reach for it.** When validation/not-found/conflict cases are handled with exceptions, or you want railway-style error handling.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Add a Result pattern and convert these handlers off exceptions."
- "Map domain errors to HTTP status codes in one place."
- "Introduce Result<T> but match the OneOf I already use."

**Get the best out of it.** Let it detect an existing `OneOf`/Result type and extend it rather than adding a competing one. Combine with `cqrs-mediatr-setup` so handlers return `Result<T>` end to end.

**Won't do.** It won't turn real infrastructure exceptions into Results — only expected, domain-level outcomes.
