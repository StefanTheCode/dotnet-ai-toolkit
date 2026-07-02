# How to use — Microservice Template Generator

**What it is.** Scaffolds a new ASP.NET Core service with production essentials wired up front: health checks (live/ready), OpenTelemetry, structured logging, validated config, Docker, and a messaging stub.

**When to reach for it.** Starting a new service, or standardizing how new services look on your platform.

**How to use it.** For a new service, name it and its responsibility. For an existing platform, point it at the repo (path or GitHub URL) to match the house template. Example prompts:
- "Scaffold an orders microservice with NATS messaging."
- "New service matching our existing services' setup."
- "Service template with health checks, OTel, Serilog, and a Dockerfile."

**Get the best out of it.** Let it read an existing service so the new one is consistent. Pair with `dockerfile-generator`, `healthchecks-setup`, and `serilog-logging-setup` for deeper config.

**Won't do.** It won't build your domain — it gives the production-ready shell; feature skills fill it in.
