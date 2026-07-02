# How to use — Authorization Policy Generator

**What it is.** Designs ASP.NET Core authorization — named policies, role/claim checks, custom requirements/handlers, and resource-based authorization (the IDOR-prevention kind).

**When to reach for it.** Restricting endpoints, replacing scattered role strings, or enforcing "can this user act on this object".

**How to use it.** Point it at your project (path or GitHub URL). Example prompts:
- "Design authorization policies for my API."
- "Restrict this endpoint to managers."
- "Stop users from accessing other users' records." (resource-based)

**Get the best out of it.** Ask for resource-based checks where ownership matters — that's what blocks IDOR. Consider a fallback policy so new endpoints aren't accidentally anonymous. Pairs with `jwt-auth-setup` and `aspnetcore-security-auditor`.

**Won't do.** It won't issue tokens — that's `jwt-auth-setup`; this is the authorization half.
