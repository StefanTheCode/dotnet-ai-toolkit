# How to use — Secrets & Config Auditor

**What it is.** Scans a .NET codebase for hardcoded secrets and unsafe config (keys/passwords/tokens in source, appsettings, pipeline YAML) and recommends a secret-store fix.

**When to reach for it.** Before open-sourcing, during a security pass, or when unsure what's safe to commit.

**How to use it.** Point it at your project or repo (path or GitHub URL). Example prompts:
- "Do I have any hardcoded secrets?"
- "Is my appsettings safe to commit?"
- "Where should these connection strings actually live?"

**Get the best out of it.** Point it at the whole repo, including config and pipeline files — that's where most leaks hide. If it finds a committed secret, rotate it (git history keeps it). Pairs with `aspnetcore-security-auditor`.

**Won't do.** It scans the working tree; deep git-history scanning is a job for a dedicated tool (gitleaks) it will recommend.
