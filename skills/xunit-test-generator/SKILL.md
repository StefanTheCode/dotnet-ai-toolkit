---
name: xunit-test-generator
description: Generate xUnit unit tests for C#/.NET code. Use this skill whenever the user shares a class, service, handler, or method and wants tests written, asks for unit tests, test coverage, "write tests for this", AAA tests, theory/data-driven tests, or mocking with Moq/NSubstitute. Always use this skill for .NET unit-test generation; it produces complete, compiling test classes with edge cases and proper mocking rather than a single example.
---

# xUnit Test Generator

Write complete, compiling xUnit test classes for the supplied C# code — happy paths, edge cases, and failure cases — using AAA structure and proper mocking.

## Input — point it at your code

You don't need pasted snippets. Accept any scope and figure out what to test yourself:

- **A single class/file** — generate tests for that type.
- **A folder** — generate (or fill gaps in) tests for every public type under it.
- **A whole project or solution** — find untested public types and prioritize the ones with real logic.
- **A GitHub URL** — clone first, then scan: `git clone --depth 1 <url> /tmp/repo && cd /tmp/repo`

Discover targets and existing coverage with Glob/Grep/Bash, e.g.:
```
grep -rln "public class\|public record\|public sealed class" --include=*.cs <target> | grep -vi "/obj/\|/bin/\|Tests"
ls **/*Tests.cs 2>/dev/null   # what's already covered / which test stack & libs they use
```
Match the project's existing test stack (mocking + assertion libs) by reading a current test file rather than asking. For a folder/project, list the types you'll cover and start with the highest-value ones. Skip trivial DTOs/POCOs. Always say which files you generated tests for.

## When this runs

The user points at code to test — a class, a folder, a project, or a GitHub link — and wants unit tests. Default to **NSubstitute + FluentAssertions**, but if the project already uses **Moq** or built-in `Assert`, match it (detect from an existing test file). Only ask if there's no existing test project and the choice is genuinely ambiguous.

## Workflow

1. **Read the unit under test.** Identify public behavior, dependencies to mock, branches, and edge cases.
2. **Enumerate test cases** before writing: happy path, each branch, boundary values, null/empty inputs, exception paths, and any business rule.
3. **Generate the test class** — one logical assertion per test, descriptive names, AAA layout, mocked dependencies.
4. **Use `[Theory]`** with `[InlineData]`/`[MemberData]` for the same logic over many inputs instead of copy-pasted `[Fact]`s.
5. **Call out gaps** — anything not unit-testable (needs integration test, hits DB/HTTP) gets flagged, not faked.

## Conventions

- **Naming:** `MethodName_Scenario_ExpectedOutcome` (e.g. `PlaceOrder_WhenCustomerHasNoCredit_ReturnsFailure`).
- **Structure:** explicit `// Arrange / // Act / // Assert`.
- **One behavior per test.** Multiple asserts are fine only when they describe one outcome.
- **No logic in tests** — no loops/conditionals deciding expected values; use `[Theory]` data instead.
- **Mock only what you own / what's a real dependency.** Don't mock value objects or the system under test.

## Example output

```csharp
public class OrderServiceTests
{
    private readonly IOrderRepository _repo = Substitute.For<IOrderRepository>();
    private readonly IUnitOfWork _uow = Substitute.For<IUnitOfWork>();
    private readonly OrderService _sut;

    public OrderServiceTests() => _sut = new OrderService(_repo, _uow);

    [Fact]
    public async Task PlaceOrder_WhenValid_PersistsAndReturnsId()
    {
        // Arrange
        var cmd = new CreateOrderCommand(CustomerId: Guid.NewGuid(), Lines: new() { new("SKU1", 2) });

        // Act
        var result = await _sut.PlaceOrderAsync(cmd, CancellationToken.None);

        // Assert
        result.IsSuccess.Should().BeTrue();
        await _repo.Received(1).AddAsync(Arg.Any<Order>(), Arg.Any<CancellationToken>());
        await _uow.Received(1).SaveChangesAsync(Arg.Any<CancellationToken>());
    }

    [Theory]
    [InlineData(0)]
    [InlineData(-1)]
    public async Task PlaceOrder_WithNonPositiveQuantity_ReturnsFailure(int qty)
    {
        var cmd = new CreateOrderCommand(Guid.NewGuid(), new() { new("SKU1", qty) });

        var result = await _sut.PlaceOrderAsync(cmd, CancellationToken.None);

        result.IsSuccess.Should().BeFalse();
        await _repo.DidNotReceive().AddAsync(Arg.Any<Order>(), Arg.Any<CancellationToken>());
    }
}
```

## Output format

```
## Test plan
<bulleted list of the cases you'll cover and why>

## Tests
<the full compiling test class>

## Not covered here (needs integration tests)
<anything that hits real I/O — flagged, not mocked into meaninglessness>
```

## Principles

- Tests document behavior — names should read like a spec.
- Cover the branches that matter, not vanity 100% coverage.
- A test that can't fail is worse than no test. Every test must have a way to go red.
- Default to NSubstitute + FluentAssertions but match the project's existing stack if shown.
