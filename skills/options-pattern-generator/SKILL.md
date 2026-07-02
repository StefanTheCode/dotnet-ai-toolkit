---
name: options-pattern-generator
description: Implement the ASP.NET Core options pattern with strongly-typed, validated configuration — IOptions/IOptionsSnapshot/IOptionsMonitor, data annotations or FluentValidation, and ValidateOnStart. Use this skill whenever the user wants strongly-typed config, the options pattern, to replace magic configuration strings, validated settings, IOptions, or asks "how do I bind config / validate settings / stop using IConfiguration everywhere". Always use this skill for options-pattern requests; it generates the options class, binding, and validation.
category: DevOps
version: 1.0.0
---

# Options Pattern Generator

Replace scattered `IConfiguration["Key"]` lookups with strongly-typed, validated options classes that fail fast at startup if misconfigured.

## Input — point it at your project
Works on a target: a **project** or **GitHub URL**. Find magic-string config to replace: `grep -rn "Configuration\[\|GetValue<\|GetSection(" --include=*.cs <target>`.

## Options class + validation
```csharp
public class JwtOptions
{
    public const string Section = "Jwt";
    [Required] public string Issuer { get; init; } = default!;
    [Required] public string Audience { get; init; } = default!;
    [Required, MinLength(32)] public string Key { get; init; } = default!;
    [Range(1, 1440)] public int AccessTokenMinutes { get; init; } = 15;
}
```
```csharp
builder.Services.AddOptions<JwtOptions>()
    .Bind(builder.Configuration.GetSection(JwtOptions.Section))
    .ValidateDataAnnotations()
    .ValidateOnStart();          // fail at startup, not at first use
```
FluentValidation variant: `IValidateOptions<JwtOptions>` for rules annotations can't express.

## Which interface to inject
- **`IOptions<T>`** — singleton, value fixed at startup. Most config.
- **`IOptionsSnapshot<T>`** — scoped, re-read per request. For values that change and you want per-request freshness.
- **`IOptionsMonitor<T>`** — singleton with change notifications. For singletons that must react to config reloads.

## Principles
- **Validate at startup (`ValidateOnStart`)** — a misconfigured app should fail to boot, not throw at 3am on first use.
- **Strongly-typed beats magic strings** — refactsafe, discoverable, testable.
- Pick the right options interface for the lifetime you need (don't inject `IOptionsSnapshot` into a singleton).
- Keep secrets out of the options *values in source* — bind them from the secret store (`secrets-config-auditor`).

## How to use it & best prompts
"Convert this config to the options pattern", "strongly-type and validate my settings", "fail at startup if config is missing", "IOptions vs IOptionsSnapshot here". Pairs with `secrets-config-auditor`.
