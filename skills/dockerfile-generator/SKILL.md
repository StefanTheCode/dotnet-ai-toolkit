---
name: dockerfile-generator
description: Generate an optimized multi-stage Dockerfile and .dockerignore for a .NET app — small final image, layer caching, non-root user, and best practices. Use this skill whenever the user wants a Dockerfile, to containerize a .NET app, a multi-stage build, smaller images, chiseled/distroless images, or asks "how do I dockerize this / Dockerfile for .NET / shrink my image". Always use this skill for Dockerfile requests; it produces a production-grade multi-stage build.
category: DevOps
version: 1.0.0
---

# Dockerfile Generator

Produce a production-grade multi-stage Dockerfile for a .NET app: fast cached builds, a small secure runtime image, and a non-root user.

## Input — point it at your project
Works on a target: a **project/solution** or **GitHub URL**. Detect the entry project and TFM: `find <target> -name "*.csproj"` and read `<TargetFramework>`. Note solution layout for correct copy/restore.

## Multi-stage Dockerfile
```dockerfile
# build
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src
COPY ["MyApp/MyApp.csproj", "MyApp/"]
RUN dotnet restore "MyApp/MyApp.csproj"          # restore as its own layer = cached unless csproj changes
COPY . .
RUN dotnet publish "MyApp/MyApp.csproj" -c Release -o /app/publish /p:UseAppHost=false

# runtime (small + non-root)
FROM mcr.microsoft.com/dotnet/aspnet:9.0-noble-chiseled AS final
WORKDIR /app
COPY --from=build /app/publish .
USER $APP_UID                                     # non-root (chiseled images set this)
EXPOSE 8080
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

## .dockerignore (don't ship junk / bloat context)
```
**/bin/
**/obj/
**/.vs/
**/.git/
**/*.user
Dockerfile*
```

## Image-size options (smallest → most compatible)
- **`-chiseled`** — minimal, no shell, non-root by default. Best default for services.
- **`-alpine`** — small, musl libc (watch native deps).
- **`-noble`/standard** — largest, most compatible.
- **AOT + `runtime-deps`** for tiny self-contained images when the app is AOT-compatible.

## Principles
- **Restore before copying source** so dependency layers cache (huge build-time win).
- **Run as non-root** — chiseled images do this for you.
- **Smallest base that runs your app** — chiseled/distroless reduce attack surface.
- Keep the build context lean with `.dockerignore`.
- Pin base image tags; don't use `latest` in production.

## How to use it & best prompts
"Dockerize this .NET app", "give me a multi-stage Dockerfile", "shrink my image / use a chiseled image", "non-root container for my service". Pairs with `cicd-pipeline-generator` and `microservice-template-generator`.
