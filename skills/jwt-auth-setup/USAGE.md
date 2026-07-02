# How to use — JWT Auth Setup

**What it is.** A skill that wires up JWT authentication and authorization in ASP.NET Core with secure defaults — token validation, a token-issuing service, revocable refresh tokens, and policy-based authorization on endpoints.

**When to reach for it.** Securing an API, adding login, protecting endpoints by role/claim, setting up refresh tokens, or validating tokens from an external identity provider.

**How to use it.** Point it at your project (a path or GitHub URL) to review/harden existing auth, or for a new setup tell it whether you're **issuing tokens yourself** (your own login) or **validating tokens from an external IdP** (Azure AD / Auth0 / Keycloak). Example prompts:
- "Add JWT auth to my API — I'll handle login myself, with refresh tokens."
- "Validate Azure AD tokens and protect my admin endpoints by role."
- "Set up policy-based authorization: admins, and a 'orders:manage' permission."

**How to get the best out of it.**
- **Be clear about issuing vs. validating** — it changes which half of the setup you need.
- **Ask for the refresh-token flow done properly** — short access tokens + hashed, revocable refresh tokens in the DB with rotation. This is where most homegrown auth goes wrong.
- **Let it set the secure defaults** (HTTPS, tight `ClockSkew`, full validation, key length) rather than disabling validation to "make it work".
- **Ask where the signing key actually goes** — it'll steer you to user-secrets/Key Vault instead of committed appsettings.

**What it won't do.** It won't store your real secrets or set up your IdP tenant — it shows the wiring and tells you where secrets belong. It's not a full Identity/user-management system; for that, it'll point you toward ASP.NET Core Identity.
