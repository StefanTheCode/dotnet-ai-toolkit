# How to use — NuGet Dependency Analyzer

**What it is.** Analyzes a multi-project solution's NuGet dependencies — version drift across projects, transitive conflicts, unused packages — and consolidates with Central Package Management.

**When to reach for it.** Version conflicts, "works on my machine" binding issues, or cleaning up dependency sprawl.

**How to use it.** Point it at your solution (path or GitHub URL). Example prompts:
- "Analyze my solution's NuGet dependencies."
- "Fix version conflicts across projects."
- "Set up Central Package Management."

**Get the best out of it.** Point it at the whole solution so it sees drift across projects. Ask for the `Directory.Packages.props` scaffold to lock versions in one place. Pairs with `dependency-vuln-scanner`.

**Won't do.** It analyzes versions/usage statically; confirm runtime behavior with a build after consolidating.
