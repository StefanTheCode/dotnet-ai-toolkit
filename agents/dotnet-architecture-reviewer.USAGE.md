# How to use — .NET Architecture Reviewer (agent)

**What it is.** An agent (not a single-shot skill) that reads a whole .NET repository, maps the project dependency graph, detects the intended architecture style, and produces a ranked report: layering/dependency-rule violations, coupling, CQRS hygiene, EF boundary leaks, testability — each with a concrete fix, plus a prioritized action list.

**When to reach for it.** A structural second opinion on a codebase, onboarding to an inherited project, a "is my architecture clean" check, or a periodic health review.

**How to use it.** Point it at a repo or solution folder. Because it's an agent, it explores files on its own — you don't need to paste code. Example prompts:
- "Review the architecture of this solution."
- "I inherited this .NET repo — map the structure and tell me what's wrong."
- "Is this following Clean Architecture properly? What violates it?"

**How to get the best out of it.**
- **Give it the whole solution**, not one file — its value is seeing the dependency graph across projects.
- **Tell it the intended style** if you know it ("this is meant to be Vertical Slice") so it reviews against the right rules; otherwise it infers.
- **Ask for the prioritized action list** to be the focus if you want a get-started-now plan rather than a full audit.
- **Run it periodically** (it pairs well with a monthly cadence) and diff the reports to track whether the structure is improving or rotting.

**What it won't do.** It reviews structure, not runtime behavior or business correctness. It's deliberately practical — it'll tell you when a "violation" is actually the right call for your team size rather than flagging dogma.
