# How to use — CI/CD Pipeline Generator

**What it is.** Generates a complete .NET CI/CD pipeline (GitHub Actions or Azure DevOps) — restore, build, test + coverage, vulnerability scan, container build, and gated deploy.

**When to reach for it.** Setting up CI for a repo, or adding test/coverage/security gates to an existing pipeline.

**How to use it.** Point it at your repo (path or GitHub URL). Example prompts:
- "Set up GitHub Actions for my .NET app."
- "Azure DevOps pipeline with tests and coverage."
- "Add a vulnerability scan that fails the build."

**Get the best out of it.** Let it detect your platform, TFM, test projects, and Dockerfile so stages fit. Ask to gate deploys with environments/approvals. Pairs with `dockerfile-generator` and `dependency-vuln-scanner`.

**Won't do.** It won't put secrets in YAML — it wires them through the platform's secret store.
