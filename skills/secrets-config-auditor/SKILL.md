---
name: secrets-config-auditor
description: Audit a .NET codebase for hardcoded secrets and unsafe configuration — connection strings, API keys, passwords, tokens in source/appsettings, and insecure config patterns — and recommend secret-store fixes. Use this skill whenever the user wants to find hardcoded secrets, leaked credentials, secrets in appsettings/source, config security, or asks "do I have secrets in my code / is my config safe / where are my credentials". Always use this skill for secrets/config audits; it scans and recommends a secret store.
category: Security
version: 1.0.0
---

# Secrets & Config Auditor

Find credentials and secrets that shouldn't be in source or committed config, and move them to a proper secret store.

## Input — point it at your project / repo
Works on a target: a **project**, **repo**, or **GitHub URL**. Scan source, config, and (if a repo) committed history hints:
```
grep -rniE "password\s*=|pwd=|apikey|api_key|secret|connectionstring|bearer |token\s*=|AccessKey|PrivateKey" \
  --include=*.cs --include=*.json --include=*.config --include=*.yml <target> | grep -vi "/obj/\|/bin/"
grep -rn "EnableSensitiveDataLogging\|//.*password\|placeholder" <target>
```
Check `appsettings*.json`, `.env`, `launchSettings.json`, pipeline YAML, and `.csproj`.

## What it flags
1. 🔴 **Hardcoded secrets in source** — keys/passwords/tokens in `.cs`.
2. 🔴 **Secrets in committed `appsettings.json`** — connection strings with passwords, API keys. (Dev secrets belong in user-secrets, prod in a vault.)
3. 🔴 **Secrets in pipeline YAML / Dockerfiles** — should be pipeline secret variables.
4. 🟡 **`EnableSensitiveDataLogging()`** unconditional — leaks parameter values to logs.
5. 🟡 **Weak config hygiene** — secrets in `launchSettings.json`, `.env` committed, no `.gitignore` coverage.

## Fix recommendations
- **Dev:** `dotnet user-secrets` (per-project, outside the repo).
- **Prod:** Azure Key Vault / AWS Secrets Manager / HashiCorp Vault, surfaced via configuration providers or env vars.
- **CI:** pipeline secret variables, never in YAML literals.
- Rotate anything that was committed — it's compromised even after removal (git history).
- Add patterns to `.gitignore`; consider a pre-commit secret scanner (gitleaks).

## Principles
- **A secret in source/committed config is a leaked secret** — and history keeps it. Rotate, don't just delete.
- Config should reference secrets, not contain them.
- Different environments, different stores — never one committed file with prod creds.

## How to use it & best prompts
"Do I have any hardcoded secrets", "scan my repo for leaked credentials", "is my appsettings safe to commit", "where should these connection strings live". Pairs with the `aspnetcore-security-auditor` agent.
