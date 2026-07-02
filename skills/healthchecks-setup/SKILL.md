---
name: healthchecks-setup
description: Add ASP.NET Core health checks with a liveness/readiness split and dependency checks (database, cache, message broker) for Kubernetes and load balancers. Use this skill whenever the user wants health checks, liveness/readiness probes, /health endpoints, dependency health, Kubernetes probes, or asks "how do I add health checks / readiness vs liveness / health endpoint". Always use this skill for health-check requests; it wires the live/ready split and dependency checks correctly.
category: Observability
version: 1.0.0
---

# Health Checks Setup

Add health checks that orchestrators can actually use: a **liveness** probe (am I alive?) separate from a **readiness** probe (can I serve traffic?), with dependency checks tagged correctly.

## Input — point it at your project
Works on a target: a **project** or **GitHub URL**. Inspect: `grep -rn "AddHealthChecks\|MapHealthChecks\|UseHealthChecks" --include=*.cs <target>` and the dependencies in `.csproj` (DB/cache/broker) to know what to check.

## Wiring (the live/ready split that matters)
```csharp
builder.Services.AddHealthChecks()
    .AddNpgSql(conn, name: "db", tags: new[] { "ready" })
    .AddRedis(redisConn, name: "redis", tags: new[] { "ready" })
    .AddCheck("self", () => HealthCheckResult.Healthy(), tags: new[] { "live" });

// Liveness: is the process up? (no dependencies — don't restart on a DB blip)
app.MapHealthChecks("/health/live", new() { Predicate = c => c.Tags.Contains("live") });

// Readiness: can it serve? (include dependencies — pull from LB while a dep is down)
app.MapHealthChecks("/health/ready", new()
{
    Predicate = c => c.Tags.Contains("ready"),
    ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse   // JSON detail
});
```

## Kubernetes probes
```yaml
livenessProbe:  { httpGet: { path: /health/live,  port: 8080 }, periodSeconds: 10 }
readinessProbe: { httpGet: { path: /health/ready, port: 8080 }, periodSeconds: 5 }
```

## Principles
- **Liveness ≠ readiness.** Liveness must NOT depend on the DB — a transient DB outage shouldn't restart your pods. Readiness should, so traffic is withheld until dependencies recover.
- Keep checks **fast and cheap** (timeouts); a slow health check causes flapping.
- Don't expose detailed health publicly without auth — it leaks topology.
- Add a check per critical dependency, tagged `ready`.

## How to use it & best prompts
"Add health checks with liveness and readiness", "wire DB and Redis health", "Kubernetes probes for my service", "why does a DB blip restart my pods" (answer: liveness depends on DB — fix the split). Pairs with `microservice-template-generator`.
