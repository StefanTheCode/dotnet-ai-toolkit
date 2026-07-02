# How to use — xUnit Test Generator

**What it is.** A skill that writes complete, compiling xUnit test classes for your C# code — happy paths, edge cases, and failure cases — with AAA structure, descriptive names, and proper mocking (NSubstitute or Moq, FluentAssertions or built-in asserts).

**When to reach for it.** New code that needs tests, raising coverage on a service/handler, or generating a test scaffold you'll refine.

**How to use it.** Point it at what you want tested — a **class/file**, a **folder**, a whole **project**, or a **GitHub URL**. For a folder or project it finds untested public types and prioritizes the ones with real logic, matching your existing test stack. Example prompts:
- "Write xUnit tests for `OrderService.cs`."
- "Generate tests for everything in `src/Core/Features/Orders/`."
- "Find untested classes in this project and write tests for the important ones."
- "Add tests for this repo: https://github.com/me/myapi"

**How to get the best out of it.**
- **Give it the dependencies' interfaces** so it mocks the right seams instead of guessing.
- **Ask for the test plan first** if it's a complex class — it lists the cases it'll cover so you can add any it missed before it writes them.
- **Point out the business rules** that matter; it'll turn them into named tests that read like a spec.
- **Prefer `[Theory]`** for repetitive cases — ask it to consolidate similar tests into data-driven ones.

**What it won't do.** It won't write meaningful tests for code that needs real I/O — it flags those for the integration-test-setup skill instead of mocking them into nonsense. It won't chase vanity 100% coverage; it targets the branches that matter.
