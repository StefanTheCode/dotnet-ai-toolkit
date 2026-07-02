# How to use — Dependency Vulnerability Scanner

**What it is.** Scans NuGet dependencies (direct + transitive) for known CVEs, deprecation, and staleness, then recommends the safest upgrade path.

**When to reach for it.** Security review, pre-release checks, or after a CVE alert.

**How to use it.** Point it at your project/solution (path or GitHub URL). Example prompts:
- "Scan my project for vulnerable packages."
- "Any CVEs in my dependencies, including transitive?"
- "This transitive package is flagged — how do I fix it?"

**Get the best out of it.** Let it run a restore so the scan is real (not version-guessing). Ask for the minimal safe bump to limit regression risk. Wire `dotnet list package --vulnerable` into CI. Pairs with `nuget-dependency-analyzer`.

**Won't do.** Offline, it can only flag known-risky versions and marks results "verify".
