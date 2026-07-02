---
name: jwt-auth-setup
description: Set up JWT authentication and authorization in ASP.NET Core. Use this skill whenever the user wants to add login/auth, issue or validate JWT tokens, set up bearer authentication, refresh tokens, role/policy-based authorization, claims, or asks "how do I secure my API / add JWT auth / protect my endpoints". Always use this skill for ASP.NET Core auth requests; it produces wired-up token issuing, validation, and authorization policies with secure defaults rather than a snippet.
---

# JWT Auth Setup (ASP.NET Core)

Wire up JWT bearer authentication and authorization in an ASP.NET Core API with secure defaults: token issuing, validation, refresh tokens, and policy-based authorization.

## Input — point it at your project

If they point at a project or repo, inspect the existing auth setup before changing anything:

- **A project or repo root** — find the auth wiring and review it.
- **A GitHub URL** — clone first, then inspect: `git clone --depth 1 <url> /tmp/repo && cd /tmp/repo`

Discover current state yourself, e.g.:
```
grep -rn "AddAuthentication\|AddJwtBearer\|TokenValidationParameters\|AddAuthorization\|RequireAuthorization\|RequireHttpsMetadata" --include=*.cs <target> | grep -vi "/obj/\|/bin/"
```
If auth already exists, **review and harden it** (validation flags, HTTPS metadata, ClockSkew, refresh-token handling, where the signing key lives) rather than re-adding from scratch, and flag any access-control gaps (endpoints without `RequireAuthorization` / no fallback policy).

## When this runs

The user wants to secure an API with JWTs, or wants their existing auth reviewed. If new: confirm whether they're **issuing** tokens themselves (own login) or **validating** tokens from an external IdP (Azure AD / Auth0 / Keycloak). Defaults below cover self-issued; for external IdP, the validation half is the same and issuing is delegated.

## Packages

```
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package System.IdentityModel.Tokens.Jwt
```

## Validation wiring (Program.cs)

```csharp
var jwt = builder.Configuration.GetSection("Jwt");

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = jwt["Issuer"],
            ValidAudience = jwt["Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(jwt["Key"]!)),
            ClockSkew = TimeSpan.FromSeconds(30) // tighten the default 5 min
        };
    });

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", p => p.RequireRole("Admin"));
    options.AddPolicy("CanManageOrders", p => p.RequireClaim("permission", "orders:manage"));
});

// ... after build:
app.UseAuthentication();
app.UseAuthorization();
```

## Token issuing service

```csharp
public class TokenService(IConfiguration config)
{
    public string CreateAccessToken(User user)
    {
        var jwt = config.GetSection("Jwt");
        var claims = new List<Claim>
        {
            new(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
            new(JwtRegisteredClaimNames.Email, user.Email),
            new(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
            new(ClaimTypes.Role, user.Role)
        };

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwt["Key"]!));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: jwt["Issuer"],
            audience: jwt["Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(15), // short-lived access token
            signingCredentials: creds);

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

## Refresh tokens (do this properly)

- Access token: **short-lived** (~15 min). Refresh token: **long-lived, persisted, revocable**.
- Store refresh tokens **hashed** in the DB with an expiry and a `Revoked` flag — never just a JWT you can't revoke.
- On refresh: validate the stored token, rotate it (issue a new refresh token, invalidate the old), then issue a new access token.
- On logout / suspected theft: revoke the refresh token row.

## Protecting endpoints

```csharp
app.MapGet("/orders", () => ...).RequireAuthorization();
app.MapPost("/orders", () => ...).RequireAuthorization("CanManageOrders");
app.MapDelete("/admin/{id}", () => ...).RequireAuthorization("AdminOnly");

// Reading the current user:
app.MapGet("/me", (ClaimsPrincipal user) =>
    Results.Ok(new { Id = user.FindFirstValue(JwtRegisteredClaimNames.Sub) }))
   .RequireAuthorization();
```

## Secure defaults (apply unless told otherwise)

- **Never put the signing key in source or appsettings committed to Git.** Use user-secrets in dev, a secret store / env var / Key Vault in prod. Key must be ≥ 32 bytes for HS256.
- **HTTPS only**; `RequireHttpsMetadata = true` in production.
- **Short access-token lifetime + revocable refresh tokens.** A JWT can't be un-issued — keep them short.
- **Validate everything** (issuer, audience, lifetime, signature). Don't disable validation to "make it work".
- **Tighten `ClockSkew`** — the 5-minute default extends token lifetime by 5 minutes.
- Prefer **policy-based authorization** over scattering `[Authorize(Roles=...)]` strings.
- Consider **asymmetric keys (RS256)** when multiple services validate tokens issued by one — they only need the public key.

## Output format

```
## What I'm setting up
<self-issued vs external IdP, what's protected>

## Packages + Program.cs wiring
...

## Token issuing / refresh
...

## Config & secrets
<appsettings shape + where the key actually goes>
```
