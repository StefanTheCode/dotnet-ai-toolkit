# How to use — Metrics Dashboard Generator

**What it is.** Generates Prometheus scrape config and a ready-to-import Grafana dashboard JSON for a .NET service — RED panels (rate/errors/duration), runtime health, and your custom meters.

**When to reach for it.** Visualizing a service's metrics, or standing up a Grafana dashboard fast.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Generate a Grafana dashboard for this service."
- "Prometheus config plus RED panels by route."
- "Add panels for my custom order metrics."

**Get the best out of it.** Let it scan for your custom meters/instruments so it builds panels for them. Verify the metric names match what your exporter emits. Pairs with `opentelemetry-setup`.

**Won't do.** It can't read your live Prometheus — confirm metric names, and watch label cardinality (no user-id labels).
