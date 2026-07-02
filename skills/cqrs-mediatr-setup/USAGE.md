# How to use — CQRS + MediatR Setup

**What it is.** Wires CQRS into an ASP.NET Core app — commands/queries, handlers, and the pipeline behaviors (validation, logging, transactions) that make CQRS worthwhile. Supports MediatR or a thin custom dispatcher.

**When to reach for it.** Adding CQRS, introducing MediatR, separating reads from writes, or standardizing a request pipeline.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Add CQRS with MediatR and a validation + transaction behavior."
- "Convert the OrdersController into commands and queries."
- "Set up a query pipeline with logging in this repo."

**Get the best out of it.** Let it read an existing feature and match your conventions. Ask for behaviors (validation/transaction/logging) — that's where CQRS pays off. Pair with `clean-architecture-scaffolder` for new slices and `result-pattern-scaffolder` for handler return types.

**Won't do.** It won't force MediatR on a tiny CRUD app — it'll tell you when a direct call is simpler.
