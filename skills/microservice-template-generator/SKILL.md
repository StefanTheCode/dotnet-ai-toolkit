---
name: microservice-template-generator
description: Generate a production-shaped ASP.NET Core microservice template — health checks, OpenTelemetry, structured logging, config/options, Docker, and a messaging consumer/producer. Use this skill whenever the user wants a new microservice, a service template/scaffold, a service starter with observability and health checks, or asks "scaffold a microservice / give me a service template". Always use this skill for microservice-template requests; it wires the production essentials, not just a Hello World.
category: Architecture
version: 1.0.0
---

# Microservice Template Generator

Scaffold a new ASP.NET Core service with the production essentials wired from day one: health checks, OpenTelemetry, structured logging, typed config, Docker, and a messaging consumer/producer — so a new service starts at "production-ready", not "Hello World".

## Input — point it at your workspace
For a new service, take the name + responsibility. For an existing platform, **match the house template**: point at the repo (path or **GitHub URL**) and read an existing service first (`find <target> -name Program.cs`, check shared infra packages) so the new one is consistent.

## What it wires
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOptions<ServiceOptions>().Bind(builder.Configuration.GetSection("Service")).ValidateOnStart();
builder.Services.AddHealthChecks().AddNpgSql(conn).AddCheck<ReadinessCheck>("ready");
builder.Host.UseSerilog((ctx, lc) => lc.ReadFrom.Configuration(ctx.Configuration).WriteTo.Console(new RenderedCompactJsonFormatter()));

builder.Services.AddOpenTelemetry()
    .ConfigureResource(r => r.AddService("orders-service"))
    .WithTracing(t => t.AddAspNetCoreInstrumentation().AddHttpClientInstrumentation().AddOtlpExporter())
    .WithMetrics(m => m.AddAspNetCoreInstrumentation().AddRuntimeInstrumentation().AddOtlpExporter());

var app = builder.Build();
app.MapHealthChecks("/health/live",  new() { Predicate = _ => false });
app.MapHealthChecks("/health/ready", new() { Predicate = c => c.Tags.Contains("ready") });
app.Run();
```
Plus: a multi-stage `Dockerfile`, `appsettings` template, a messaging consumer/producer stub (MassTransit/NATS/raw), and a `README` for the service.

## Folder shape
```
OrdersService/
├── Program.cs
├── Endpoints/        ├── Domain/        ├── Infrastructure/
├── Dockerfile        ├── appsettings.json
└── OrdersService.csproj
```

## Principles
- **Production essentials are not optional:** health (live/ready split), telemetry, structured logs, validated config — wired before the first endpoint.
- **Match the platform.** A new service should look like the others; read one first.
- **12-factor config** — environment/secret store, not committed appsettings.
- Don't gold-plate: include what every service needs, leave domain-specific bits to the feature skills.

## How to use it & best prompts
"Scaffold an orders microservice with NATS messaging", "give me a service template matching our existing services", "new service with health checks, OTel, Serilog, Docker". Pair with `dockerfile-generator`, `healthchecks-setup`, `serilog-logging-setup` for deeper config.
