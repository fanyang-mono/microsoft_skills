# Authentication — .NET Library Quick Reference

> Condensed from **azure-identity-dotnet**. Full patterns (ASP.NET Core DI,
> sovereign clouds, brokered auth, certificate credentials)
> in the **azure-identity-dotnet** plugin skill if installed.

## Install

```bash
dotnet add package Azure.Identity
```

## Quickstart

> **Auth:** `DefaultAzureCredential` is for local development. See [auth-best-practices.md](../auth-best-practices.md) for production patterns.

```csharp
using Azure.Identity;

var credential = new DefaultAzureCredential();
```

## Best Practices

- Use `DefaultAzureCredential` for **local development only**. In production, use deterministic credentials (`ManagedIdentityCredential`) — see [auth-best-practices.md](../auth-best-practices.md)
- Reuse credential instances — single instance shared across clients
- Azure CLI for local dev — run `az login` before running your app
- Least privilege — grant only required permissions to service principals
- Configure retry policies for credential operations
- Enable logging with `AzureEventSourceListener` for debugging auth issues
- Environment variables — use for CI/CD, not hardcoded secrets
