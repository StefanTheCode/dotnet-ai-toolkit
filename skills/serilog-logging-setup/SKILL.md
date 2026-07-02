---
name: serilog-logging-setup
description: Set up structured logging with Serilog in ASP.NET Core — JSON output, enrichers, request logging, log levels, and sinks — correlated with traces. Use this skill whenever the user wants Serilog, structured logging, JSON logs, request logging, log enrichment, log sinks (Seq/Elastic/console), or asks "how do I add logging / set up Serilog / get structured logs". Always use this skill for Serilog/structured-logging requests; it wires production-shaped logging, not just Console.WriteLine.
category: Observability
version: 1.0.0
---

# Serilog Structured Logging Setup

Wire up Serilog in ASP.NET Core for structured (JSON) logs with enrichers, request logging, sensible levels, and sinks — correlated with your traces.

## Input — point it at your project
Works on a target: a **project** or **GitHub URL**. Inspect current logging: `grep -rn "UseSerilog\|ILogger\|Log.Logger\|AddSerilog" --include=*.cs <target>`. If OTel is present, wire log/trace correlation.

## Packages + wiring
```
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Seq        # or Console/Elastic/etc.
```
```csharp
builder.Host.UseSerilog((ctx, services, lc) => lc
    .ReadFrom.Configuration(ctx.Configuration)
    .ReadFrom.Services(services)
    .Enrich.FromLogContext()
    .Enrich.WithMachineName()
    .Enrich.WithProperty("service", "orders")
    .WriteTo.Console(new RenderedCompactJsonFormatter())   // structured JSON
    .WriteTo.Seq(ctx.Configuration["Seq:Url"]!));

app.UseSerilogRequestLogging();   // one tidy log line per request instead of the default noise
```

## Structured logging done right
```csharp
// GOOD: message template + properties (queryable)
logger.LogInformation("Order {OrderId} placed by {CustomerId} for {Total}", id, customerId, total);
// BAD: interpolated string (loses structure)
logger.LogInformation($"Order {id} placed");
```

## What to set
- **Levels** — `Information` default, `Warning` for noisy frameworks, `Debug` only in dev. Configure via `appsettings`.
- **Enrichers** — machine, environment, and a correlation/trace id (see `correlation-id-middleware`).
- **Request logging** — `UseSerilogRequestLogging` for one structured line per request with status + duration.
- **Trace correlation** — include `TraceId`/`SpanId` so logs link to traces.

## Principles
- **Message templates, not interpolation** — structure is the whole point of structured logging.
- **Don't log secrets/PII.** Be deliberate about what goes into properties.
- Log at the right level; `Information` for business events, `Error` with the exception object (not `ex.Message`).
- One sink for local (console JSON), one for aggregation (Seq/Elastic/Loki).

## How to use it & best prompts
"Set up Serilog with JSON logs and Seq", "add request logging", "correlate my logs with traces", "fix my logging to be structured". Pairs with `opentelemetry-setup` and `correlation-id-middleware`.
