---
name: metrics-dashboard-generator
description: Generate Prometheus scrape config and a Grafana dashboard JSON for a .NET service's metrics — RED/USE method panels for requests, errors, latency, GC, and custom meters. Use this skill whenever the user wants a Grafana dashboard, Prometheus config, metrics panels, RED/USE dashboards, to visualize .NET metrics, or asks "how do I dashboard my metrics / Grafana for my service". Always use this skill for metrics-dashboard requests; it emits ready-to-import config.
category: Observability
version: 1.0.0
---

# Metrics Dashboard Generator

Produce ready-to-import Prometheus scrape config and a Grafana dashboard JSON for a .NET service — built around the RED method (Rate, Errors, Duration) plus runtime health and your custom meters.

## Input — point it at your project
Works on a target: a **project** or **GitHub URL**. Find which metrics exist: `grep -rn "AddPrometheusExporter\|MapPrometheusScrapingEndpoint\|Meter(\|CreateCounter\|CreateHistogram\|AddOpenTelemetry" --include=*.cs <target>`. Note custom meter/instrument names to add panels for them.

## Prometheus scrape
```yaml
scrape_configs:
  - job_name: 'orders-service'
    metrics_path: /metrics
    static_configs:
      - targets: ['orders-service:8080']
```
Service side: `OpenTelemetry ... .WithMetrics(m => m.AddPrometheusExporter())` + `app.MapPrometheusScrapingEndpoint();`.

## Grafana dashboard (panels generated)
- **Rate** — requests/sec: `rate(http_server_request_duration_seconds_count[1m])` by route.
- **Errors** — 5xx ratio: `sum(rate(...{http_response_status_code=~"5.."}[5m])) / sum(rate(...[5m]))`.
- **Duration** — p50/p95/p99: `histogram_quantile(0.95, sum(rate(http_server_request_duration_seconds_bucket[5m])) by (le, route))`.
- **Runtime** — GC collections, heap size, thread pool queue, working set (from `AddRuntimeInstrumentation`).
- **Custom meters** — a panel per custom counter/histogram you detected.

Output is a complete dashboard `.json` to import into Grafana, plus the scrape snippet.

## Principles
- **RED for request services, USE for resources** — start there before vanity panels.
- Use **histogram quantiles** for latency, not averages (averages hide tail latency).
- Label by route/status, but watch **cardinality** — don't label by user id or raw path.
- Match metric names to the OTel/Prometheus conventions your exporter emits (verify the names).

## How to use it & best prompts
"Generate a Grafana dashboard for my service", "Prometheus config + RED panels", "dashboard my custom order metrics", "p95 latency panel by route". Pairs with `opentelemetry-setup`.
