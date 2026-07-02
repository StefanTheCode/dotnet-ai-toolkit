# How to use — Integration Test Setup

**What it is.** A skill that stands up real integration tests for an ASP.NET Core API — a custom `WebApplicationFactory`, a real database via Testcontainers (Postgres or SQL Server), state reset with Respawn, and a sample test that drives the full HTTP pipeline.

**When to reach for it.** When unit tests aren't enough — you need to test routing, DI, middleware, auth, EF mappings, and migrations against a real database.

**How to use it.** Point it at your API project (a path or GitHub URL); it finds `Program.cs`, the endpoints, and any existing test project, and reviews/extends rather than duplicating. Tell it your **database** (default Postgres via Testcontainers) and roughly what your API looks like. Example prompts:
- "Set up integration tests for my ASP.NET Core API against a real Postgres."
- "I need to test my orders endpoints end-to-end with a real SQL Server."
- "Add a WebApplicationFactory and one passing test for the /customers endpoint."

**How to get the best out of it.**
- **Have Docker running** — Testcontainers spins up a real DB container, which is the whole point (the EF in-memory provider lies about constraints and SQL).
- **Mention your `Program.cs` style** — with top-level statements you need `public partial class Program { }`; the skill reminds you but confirm it's there.
- **Ask for the Respawn reset hook** so tests stay independent without restarting containers (keeps the suite fast).
- **Give it a representative endpoint** and it'll produce a test you can copy for the rest.

**What it won't do.** It won't replace unit tests — don't integration-test pure logic that a fast unit test already covers. It assumes a container runtime is available in your test environment/CI.
