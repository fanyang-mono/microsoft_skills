# Authentication — Go Library Quick Reference

> Condensed from **azure-identity-go**. Full patterns (workload identity,
> certificate auth, device code, sovereign clouds)
> in the **azure-identity-go** plugin skill if installed.

## Install

```bash
go get -u github.com/Azure/azure-sdk-for-go/sdk/azidentity
```

## Quickstart

> **Auth:** `DefaultAzureCredential` is best for prototyping or local development. See [auth-best-practices.md](../auth-best-practices.md) for production patterns.

```go
import (
    "log"

    "github.com/Azure/azure-sdk-for-go/sdk/azidentity"
)

cred, err := azidentity.NewDefaultAzureCredential(nil)
if err != nil {
    log.Fatal(err)
}
```

## Best Practices

- Use `DefaultAzureCredential` for **local development only** (CLI, PowerShell, VS Code). In production, use `ManagedIdentityCredential` — see [auth-best-practices.md](../auth-best-practices.md)
- Managed identity in production — no secrets to manage, automatic rotation
- Azure CLI for local dev — run `az login` before running your app
- Least privilege — grant only required permissions to service principals
- Token caching — enabled by default, reduces auth round-trips
- Environment variables — use for CI/CD, not hardcoded secrets
