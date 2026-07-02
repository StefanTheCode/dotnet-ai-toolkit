# How to use — Options Pattern Generator

**What it is.** Implements the ASP.NET Core options pattern — strongly-typed, validated config classes with `ValidateOnStart`, and the right `IOptions`/`IOptionsSnapshot`/`IOptionsMonitor` choice.

**When to reach for it.** Replacing magic `IConfiguration["..."]` lookups, or you want config that fails fast when misconfigured.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Convert this config to the options pattern with validation."
- "Fail at startup if these settings are missing."
- "IOptions vs IOptionsSnapshot for this — which do I use?"

**Get the best out of it.** Ask for `ValidateOnStart` so misconfig stops boot, not a 3am request. Let it pick the options interface by lifetime. Keep secret *values* in a store, not in source. Pairs with `secrets-config-auditor`.

**Won't do.** It won't embed secrets in the options class — those bind from configuration/secret store.
