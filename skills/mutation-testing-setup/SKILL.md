---
name: mutation-testing-setup
description: Set up mutation testing with Stryker.NET to measure whether tests actually catch bugs, and interpret the mutation score and surviving mutants. Use this skill whenever the user wants mutation testing, Stryker, to know if their tests are any good, test quality beyond coverage, surviving mutants, or asks "are my tests actually testing anything / mutation score / Stryker setup". Always use this skill for mutation-testing requests; it configures Stryker and explains the results.
category: Testing
version: 1.0.0
---

# Mutation Testing Setup (Stryker.NET)

Measure whether your tests actually catch bugs — not just execute code — with Stryker.NET, and turn surviving mutants into better tests.

## Input — point it at your project
Works on a target: a **project/test project** or **GitHub URL**. Find the test project: `find <target> -iname "*.Tests.csproj"`.

## Setup
```
dotnet tool install -g dotnet-stryker
cd path/to/Tests.Project
dotnet stryker
```
`stryker-config.json` (scope it so runs are fast):
```json
{
  "stryker-config": {
    "project": "MyApp.csproj",
    "test-projects": ["MyApp.Tests.csproj"],
    "mutate": ["**/Core/**/*.cs", "!**/Migrations/**"],
    "thresholds": { "high": 80, "low": 60, "break": 50 }
  }
}
```

## Reading results
- **Mutation score** = killed mutants / total. Higher = tests catch more injected bugs.
- **Surviving mutant** = Stryker changed the code (e.g. flipped `>` to `>=`, removed a statement) and **no test failed** → a real gap in your assertions.
- **Timeout/no-coverage** mutants = code not exercised at all (a coverage gap, see `test-coverage-gap-finder`).

For each meaningful survivor: add or strengthen an assertion so the mutation would be caught. A test that runs code without asserting its effect lets mutants survive.

## Principles
- **Coverage says code ran; mutation says tests would catch a bug.** Mutation is the truer quality signal.
- Scope mutation runs to important code (it's slow) — don't mutate the whole solution every CI run.
- Don't chase 100% — focus survivors in critical logic.
- Treat survivors as "missing assertion", not "add a test for coverage".

## How to use it & best prompts
"Set up Stryker mutation testing", "are my tests actually catching bugs", "explain these surviving mutants", "improve my tests so these mutants die". Pairs with `test-coverage-gap-finder` and `xunit-test-generator`.
