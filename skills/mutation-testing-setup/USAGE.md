# How to use — Mutation Testing Setup (Stryker.NET)

**What it is.** Sets up Stryker.NET mutation testing to measure whether your tests actually catch bugs (not just run code), and interprets the score and surviving mutants.

**When to reach for it.** You have tests but want to know if they're any good — quality beyond coverage %.

**How to use it.** Point it at your test project (path or GitHub URL). Example prompts:
- "Set up Stryker mutation testing for this project."
- "Are my tests actually catching bugs?"
- "Explain these surviving mutants and fix my tests."

**Get the best out of it.** Scope the `mutate` config to important code — Stryker is slow on a whole solution. Treat each survivor as a missing assertion. Pairs with `test-coverage-gap-finder` and `xunit-test-generator`.

**Won't do.** It won't chase a 100% mutation score — it focuses survivors in critical logic.
