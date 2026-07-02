---
name: codebase-health-dashboard
description: Run the whole toolkit against a .NET repo and produce a single scored "health dashboard" - an overall score plus per-category scores (security, architecture, EF/database, performance, observability, testing, dependencies, code quality), each with findings and the top fixes. Use this skill whenever the user wants a codebase health check, an overall code quality score, a full audit of their .NET project, "how healthy is my codebase", a one-page report card, or to track their codebase score over time. Always use this skill for whole-repo health/scorecard requests; it orchestrates the auditor tools, scores them with a fixed rubric, and writes a self-contained dashboard.
category: AI Tooling
version: 1.0.0
---

# Codebase Health Dashboard

Point this at a .NET repo and get back a **single scored report card**: one overall health score, eight category scores, the findings behind each score, and the highest-leverage fixes. It is the "run everything and tell me where I stand" tool.

## Input - point it at your project
Works on a target: a **project/solution** or a **GitHub URL** (`git clone --depth 1 <url>` then scan). Find the surface first:
```
find <target> -name "*.sln" -o -name "*.csproj" | grep -vi "/obj/\|/bin/"
```

## Workflow
1. **Run the relevant auditors** on the repo and collect their findings. Use the installed tools if present, otherwise apply their checklists directly:
   - **Security** - `aspnetcore-security-auditor`, `secrets-config-auditor`
   - **Architecture** - `dotnet-architecture-reviewer`
   - **EF / Database** - `db-performance-auditor`, `ef-core-query-optimizer`
   - **Performance** - `async-await-auditor`, `memory-allocation-analyzer`
   - **Observability** - `observability-gap-finder`
   - **Testing** - `test-coverage-gap-finder`
   - **Dependencies** - `dependency-vuln-scanner`, `nuget-dependency-analyzer`
   - **Code Quality** - `dotnet-code-reviewer`
2. **Score each category** with the fixed rubric below (deterministic, so re-runs are comparable).
3. **Fill the template** `references/health-dashboard-template.html` - replace `/*__HEALTH_DATA__*/` with the JSON object (shape below) and write it to `health-dashboard/index.html`.
4. **Save a history record** to `health/health-<YYYY-MM-DD>.json` with the same data, so the score can be tracked over time.
5. **Tell the user the overall score and the top 3 actions** to raise it the fastest.

## Scoring rubric (deterministic)
Each category starts at **100**. Subtract per finding by severity, floor at 0:
- critical: **-25**
- high: **-12**
- medium: **-5**
- low: **-2**

If a category genuinely has no applicable surface (e.g. no messaging, so some observability checks do not apply), score what exists and note it - do not invent findings to fill it.

**Overall** = rounded average of the category scores.

**Verdict bands** (per category and overall):
- 85-100: **Strong**
- 70-84: **Healthy**
- 50-69: **Needs work**
- 0-49: **At risk**

## Output data shape (inject into the template)
```json
{
  "project": "Your repo (or anonymized name)",
  "date": "2026-06-22",
  "overall": 73,
  "summary": "One paragraph: the headline, the biggest risk, the quickest win.",
  "categories": [
    {
      "name": "Security",
      "score": 61,
      "verdict": "Needs work",
      "findings": [
        { "severity": "critical", "title": "1 of 75 endpoints require authorization", "location": "endpoints" },
        { "severity": "high", "title": "Sensitive data logging enabled in all environments", "location": "DbContext setup" }
      ]
    }
    // ... one object per category that applies
  ],
  "topActions": [
    "Add a fallback authorization policy and audit endpoint coverage.",
    "Gate EnableSensitiveDataLogging behind IsDevelopment().",
    "Push the in-memory filtering in the orders query into SQL."
  ]
}
```

## Principles
- **Deterministic scoring** so the number means something and members can watch it climb week to week.
- **Honest** - credit what is healthy; do not pad findings to lower a score artificially.
- **Actionable** - every category links its score to specific findings and the fix, and the top 3 actions are the fastest wins.
- **Private** - it runs on the target the user points at; nothing leaves their machine.

## How to use it & best prompts
"Run a health check on this repo", "give my .NET project an overall score", "audit my whole solution and show me a report card", "how healthy is this codebase". Re-run it weekly and watch the score move. Pairs with every auditor agent and the `toolkit-dashboard`.
