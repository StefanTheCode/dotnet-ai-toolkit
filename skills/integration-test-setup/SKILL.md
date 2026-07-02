---
name: integration-test-setup
description: Set up integration tests for an ASP.NET Core app using WebApplicationFactory and Testcontainers. Use this skill whenever the user wants integration tests, end-to-end API tests, tests that hit a real database, WebApplicationFactory setup, Testcontainers, test fixtures with a real Postgres/SQL Server/Redis, or asks "how do I integration-test my API / test against a real database". Always use this skill for .NET integration-testing requests; it produces a working test project, factory, and a sample passing test.
---

# Integration Test Setup (.NET)

Stand up real integration tests for an ASP.NET Core API: a custom `WebApplicationFactory`, a real database via Testcontainers, and a sample test that exercises the full HTTP pipeline.

## Input — point it at your project

Point this at the API project or repo, not a pasted file — inspect what exists first:

- **A project or repo root** — find `Program.cs`, the endpoints, and any existing integration-test project.
- **A GitHub URL** — clone first, then inspect: `git clone --depth 1 <url> /tmp/repo && cd /tmp/repo`

Discover the surface yourself before generating, e.g.:
```
find <target> -name Program.cs | grep -vi "/obj/\|/bin/"
grep -rn "MapGet\|MapPost\|MapPut\|MapDelete\|: ControllerBase\|ICarterModule" --include=*.cs <target> | head
find <target> -iname "*IntegrationTest*" -o -iname "*.IntegrationTests.csproj"
```
If an integration-test project already exists, **review and extend it** (e.g. flag `EnsureDbCreated` vs `MigrateAsync`, unpinned images, missing state reset) instead of scaffolding from scratch. Pick a representative endpoint to produce the first test.

## When this runs

The user wants tests that exercise the app for real — routing, DI, middleware, EF Core against an actual database — not mocked units. Inspect the project, then confirm the database (default **PostgreSQL via Testcontainers**; support SQL Server) only if you can't infer it from the existing setup.

## Packages

```
dotnet add package Microsoft.AspNetCore.Mvc.Testing
dotnet add package Testcontainers.PostgreSql   # or Testcontainers.MsSql
dotnet add package xunit
dotnet add package FluentAssertions
dotnet add package Respawn                      # reset DB state between tests
```

## The factory (real DB swapped in)

```csharp
public class IntegrationTestFactory : WebApplicationFactory<Program>, IAsyncLifetime
{
    private readonly PostgreSqlContainer _db = new PostgreSqlBuilder()
        .WithImage("postgres:16-alpine")
        .Build();

    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureTestServices(services =>
        {
            // Remove the app's registered DbContext and point it at the container
            services.RemoveAll<DbContextOptions<AppDbContext>>();
            services.AddDbContext<AppDbContext>(o => o.UseNpgsql(_db.GetConnectionString()));
        });
    }

    public async Task InitializeAsync()
    {
        await _db.StartAsync();
        using var scope = Services.CreateScope();
        var ctx = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        await ctx.Database.MigrateAsync();
    }

    public new async Task DisposeAsync() => await _db.DisposeAsync();
}
```

> `Program` must be reachable — add `public partial class Program { }` at the bottom of the API's `Program.cs` if using top-level statements.

## Shared collection (one container for the test class/collection)

```csharp
[CollectionDefinition(nameof(IntegrationCollection))]
public class IntegrationCollection : ICollectionFixture<IntegrationTestFactory> { }
```

## Sample test

```csharp
[Collection(nameof(IntegrationCollection))]
public class OrdersApiTests
{
    private readonly HttpClient _client;
    public OrdersApiTests(IntegrationTestFactory factory) => _client = factory.CreateClient();

    [Fact]
    public async Task CreateOrder_ReturnsCreated_AndPersists()
    {
        var request = new { customerId = Guid.NewGuid(), lines = new[] { new { sku = "SKU1", qty = 2 } } };

        var response = await _client.PostAsJsonAsync("/orders", request);

        response.StatusCode.Should().Be(HttpStatusCode.Created);
        var id = await response.Content.ReadFromJsonAsync<Guid>();
        id.Should().NotBeEmpty();

        var getResponse = await _client.GetAsync($"/orders/{id}");
        getResponse.StatusCode.Should().Be(HttpStatusCode.OK);
    }
}
```

## State reset between tests (Respawn)

To keep tests independent without restarting the container, reset tables between tests:

```csharp
private Respawner _respawner = default!;
// after migrate:
_respawner = await Respawner.CreateAsync(connString,
    new RespawnerOptions { DbAdapter = DbAdapter.Postgres });
// in a reset hook:
await _respawner.ResetAsync(connString);
```

## Principles

- **Test the seams that mocks can't reach:** routing, model binding, auth, EF mappings, migrations.
- **Real DB beats in-memory provider.** The EF Core in-memory provider doesn't enforce constraints or translate SQL the same way — Testcontainers gives you the real engine.
- **Keep tests independent** — reset state (Respawn) rather than depending on order.
- **One container per collection**, not per test, or the suite crawls. Reuse, reset state.
- Don't integration-test pure logic that a unit test already covers — integration tests are slower; spend them where they buy confidence.
