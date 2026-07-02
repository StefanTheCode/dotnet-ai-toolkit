---
name: opentelemetry-setup
description: Add or fix OpenTelemetry observability (traces, metrics, logs) in an ASP.NET Core / .NET app. Use this skill whenever the user wants to instrument a .NET service, add distributed tracing, export to Jaeger/Tempo/Prometheus/an OTLP collector/Azure Monitor, create custom spans or metrics, set up correlation across services, or asks "how do I add observability/telemetry/tracing to my .NET app". Always use this skill for OpenTelemetry/.NET observability requests; it generates wired-up, production-shaped configuration rather than a snippet.
---

# OpenTelemetry Setup (.NET)

Wire up OpenTelemetry in an ASP.NET Core or .NET worker app — traces, metrics, and logs — exported to the backend the user wants, with sensible production defaults.

## Input — point it at your project

Don't ask the user to paste `Program.cs` — point this at the project and inspect what's already there:

- **A project or repo root** — find `Program.cs` / `Startup.cs` and any existing OTel wiring.
- **A folder or single file** — if they point directly at the startup file.
- **A GitHub URL** — clone first, then inspect: `git clone --depth 1 <url> /tmp/repo && cd /tmp/repo`

Discover the current state yourself before generating, e.g.:
```
find <target> -name Program.cs -o -name Startup.cs | grep -vi "/obj/\|/bin/"
grep -rn "AddOpenTelemetry\|WithTracing\|WithMetrics\|AddOtlpExporter\|UseSerilog" --include=*.cs <target>
```
If OTel is already partly set up (e.g. metrics but no tracing), **extend it** rather than starting over, and call out the specific gap. Read the `.csproj` to see which instrumentation packages are already referenced.

## When this runs

The user wants observability in a .NET app: distributed tracing, metrics, structured logs, correlation across services, or export to a specific backend. Inspect the project first, then establish three things (ask only what you can't infer):

1. **App type** — ASP.NET Core API, worker/background service, or library.
2. **Backend** — OTLP collector (vendor-neutral, default), Jaeger, Prometheus + Grafana/Tempo, Azure Monitor / Application Insights, or "just console for now".
3. **.NET version** — defaults assume .NET 8/9.

## Workflow

1. Confirm app type, backend, and signals wanted (traces / metrics / logs — default all three).
2. Generate the `Program.cs` wiring + required NuGet packages.
3. Add resource attributes (service name, version, environment) — non-negotiable for usable telemetry.
4. Add the relevant instrumentation libraries (ASP.NET Core, HttpClient, EF Core, runtime).
5. Show how to emit a custom span and a custom metric.
6. Give the export config for the chosen backend, plus the collector/compose snippet if relevant.

## Packages

Base:
```
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Instrumentation.AspNetCore
dotnet add package OpenTelemetry.Instrumentation.Http
dotnet add package OpenTelemetry.Instrumentation.Runtime
dotnet add package OpenTelemetry.Exporter.OpenTelemetryProtocol
```
Add as needed: `OpenTelemetry.Instrumentation.EntityFrameworkCore`, `OpenTelemetry.Exporter.Prometheus.AspNetCore`, `Azure.Monitor.OpenTelemetry.AspNetCore`.

## Program.cs wiring (OTLP default)

```csharp
var builder = WebApplication.CreateBuilder(args);

const string serviceName = "my-service";
const string serviceVersion = "1.0.0";

var resource = ResourceBuilder.CreateDefault()
    .AddService(serviceName, serviceVersion: serviceVersion)
    .AddAttributes(new KeyValuePair<string, object>[]
    {
        new("deployment.environment", builder.Environment.EnvironmentName)
    });

builder.Services.AddOpenTelemetry()
    .ConfigureResource(r => r
        .AddService(serviceName, serviceVersion: serviceVersion)
        .AddAttributes(new[] {
            new KeyValuePair<string, object>("deployment.environment", builder.Environment.EnvironmentName)
        }))
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation(o => o.RecordException = true)
        .AddHttpClientInstrumentation()
        .AddEntityFrameworkCoreInstrumentation(o => o.SetDbStatementForText = true)
        .AddSource(serviceName) // your custom ActivitySource
        .SetSampler(new ParentBasedSampler(new TraceIdRatioBasedSampler(1.0))) // tune ratio in prod
        .AddOtlpExporter())
    .WithMetrics(metrics => metrics
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddRuntimeInstrumentation()
        .AddMeter(serviceName) // your custom Meter
        .AddOtlpExporter());

// Logs through OpenTelemetry
builder.Logging.AddOpenTelemetry(logging =>
{
    logging.SetResourceBuilder(resource);
    logging.IncludeScopes = true;
    logging.IncludeFormattedMessage = true;
    logging.AddOtlpExporter();
});

var app = builder.Build();
```

OTLP endpoint is configured via env var (preferred — keeps it out of code):
```
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
OTEL_EXPORTER_OTLP_PROTOCOL=grpc
```

## Custom span

```csharp
public class OrderService
{
    private static readonly ActivitySource Activity = new("my-service");

    public async Task PlaceOrderAsync(Order order, CancellationToken ct)
    {
        using var span = Activity.StartActivity("PlaceOrder");
        span?.SetTag("order.id", order.Id);
        span?.SetTag("order.total", order.Total);
        try
        {
            await _repo.SaveAsync(order, ct);
        }
        catch (Exception ex)
        {
            span?.SetStatus(ActivityStatusCode.Error, ex.Message);
            throw;
        }
    }
}
```

## Custom metric

```csharp
public class OrderMetrics
{
    private readonly Counter<long> _ordersPlaced;
    public OrderMetrics(IMeterFactory meterFactory)
    {
        var meter = meterFactory.Create("my-service");
        _ordersPlaced = meter.CreateCounter<long>("orders.placed", unit: "{orders}");
    }
    public void OrderPlaced(string region) =>
        _ordersPlaced.Add(1, new KeyValuePair<string, object?>("region", region));
}
```

## Backend variants

- **Prometheus:** add `OpenTelemetry.Exporter.Prometheus.AspNetCore`, replace metrics exporter with `.AddPrometheusExporter()`, and `app.MapPrometheusScrapingEndpoint();`.
- **Azure Monitor:** `builder.Services.AddOpenTelemetry().UseAzureMonitor();` (replaces exporters; set `APPLICATIONINSIGHTS_CONNECTION_STRING`).
- **Jaeger / Tempo:** they accept OTLP directly — point `OTEL_EXPORTER_OTLP_ENDPOINT` at the collector.

A ready-to-run collector + Jaeger + Prometheus docker-compose is in `references/otel-collector-compose.md`.

## Production defaults (apply unless told otherwise)

- **Sampling:** don't trace 100% in prod. Use `ParentBasedSampler` + a ratio (start 10–20%) or tail sampling at the collector.
- **Resource attributes:** always set `service.name`, `service.version`, `deployment.environment`. Without them telemetry is unusable across services.
- **`RecordException = true`** on ASP.NET Core spans so errors attach to traces.
- **Don't log secrets/PII** as span tags. Be careful with `SetDbStatementForText` (may capture parameters).
- **Correlation:** trace context propagates automatically over HttpClient; for messaging/queues, inject/extract `TraceContext` manually.

## Output format

```
## What I'm setting up
<app type, backend, signals>

## Packages
<dotnet add package ...>

## Program.cs
<wiring>

## Custom instrumentation example
<span + metric relevant to their domain>

## Run it
<env vars / compose / how to see the data>
```
