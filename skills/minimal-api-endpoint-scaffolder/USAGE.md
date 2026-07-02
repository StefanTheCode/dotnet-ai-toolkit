# How to use — Minimal API Endpoint Scaffolder

**What it is.** Generates ASP.NET Core Minimal API endpoints (REPR pattern) with validation, `TypedResults`, route grouping, and OpenAPI metadata — matching your project's existing endpoint style.

**When to reach for it.** Adding endpoints, converting controllers to minimal APIs, or standardizing route structure.

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Add a minimal API endpoint to create an order."
- "Convert OrdersController to minimal APIs."
- "Group these routes under /orders with auth and tags."

**Get the best out of it.** Let it read an existing endpoint first so the new one fits (Carter vs MapGroup vs MediatR). Ask for `TypedResults` and validation — that's the clean default.

**Won't do.** It won't put business logic in the endpoint — rules go to the handler/domain.
