# How to use — Health Checks Setup

**What it is.** Adds ASP.NET Core health checks with a proper liveness/readiness split and dependency checks (DB, cache, broker) for Kubernetes and load balancers.

**When to reach for it.** Adding `/health` endpoints, k8s probes, or fixing pods that restart on dependency blips.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Add liveness and readiness health checks."
- "Wire DB and Redis into readiness."
- "Why does a DB outage restart my pods?" (liveness depends on DB — it fixes the split.)

**Get the best out of it.** Let it read your `.csproj` to add the right dependency checks. Keep liveness dependency-free — that's the key lesson. Pairs with `microservice-template-generator`.

**Won't do.** It won't expose detailed health publicly without auth — that leaks topology.
