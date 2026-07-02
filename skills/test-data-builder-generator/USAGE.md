# How to use — Test Data Builder Generator

**What it is.** Generates fluent test data builders and object mothers so tests create valid domain objects in one line, with only the relevant fields made explicit.

**When to reach for it.** Repetitive, brittle `new X { ... }` setup across tests, or objects with invariants that are painful to construct.

**How to use it.** Point it at the types (path or GitHub URL). Example prompts:
- "Generate a test data builder for Order."
- "Reduce the setup boilerplate in these tests."
- "Object mothers for my domain types — builders or AutoFixture?"

**Get the best out of it.** Let it read existing tests to see the real setup pain. Builders should respect your factories/invariants — ask it to build through them. Pairs with `xunit-test-generator`.

**Won't do.** It won't bypass domain invariants to fabricate invalid objects.
