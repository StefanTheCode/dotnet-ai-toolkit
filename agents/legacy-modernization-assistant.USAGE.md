# How to use — Legacy Modernization Assistant (agent)

**What it is.** An agent that assesses a legacy .NET codebase and produces a pragmatic, incremental modernization roadmap (strangler-fig, not big-bang) — platform, architecture, DI, async, testability — sequenced by risk and value.

**When to reach for it.** Modernizing legacy code, migrating off .NET Framework, or "where do I even start with this old codebase".

**How to use it.** Point it at the codebase (path or GitHub URL). Example prompts:
- "Assess this legacy app and give me a modernization roadmap."
- "We're on .NET Framework — how do we move to modern .NET incrementally?"
- "Where do I start untangling this monolith?"

**Get the best out of it.** Let it sequence by risk/value and insist on tests before risky refactors. It hands work to `dotnet-upgrade-agent`, `modular-monolith-generator`, `test-coverage-gap-finder`, and `async-await-auditor`.

**Won't do.** It won't recommend a big-bang rewrite unless there's strong justification — incremental is the default.
