# How to use — Dockerfile Generator

**What it is.** Generates a production-grade multi-stage Dockerfile and `.dockerignore` for a .NET app — cached builds, small secure runtime image (chiseled/distroless), non-root user.

**When to reach for it.** Containerizing a service, or shrinking/hardening an existing image.

**How to use it.** Point it at your project/solution (path or GitHub URL). Example prompts:
- "Dockerize this .NET app with a multi-stage build."
- "Shrink my image — use a chiseled base."
- "Non-root container for this service."

**Get the best out of it.** Let it detect the entry project and TFM so the copy/restore paths are right. Ask for chiseled as the default. Pairs with `cicd-pipeline-generator` and `microservice-template-generator`.

**Won't do.** It won't use `latest` base tags in production — it pins versions.
