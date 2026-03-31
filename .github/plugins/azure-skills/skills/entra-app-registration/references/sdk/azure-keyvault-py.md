# Key Vault — Python Library Quick Reference

> Condensed from **azure-keyvault-py**. Full patterns (async clients,
> cryptographic operations, certificate management, error handling)
> in the **azure-keyvault-py** plugin skill if installed.

## Install

```bash
pip install azure-keyvault-secrets azure-keyvault-keys azure-keyvault-certificates azure-identity
```

## Quickstart

> **Auth:** `DefaultAzureCredential` is best for prototyping or local development. See [auth-best-practices.md](../auth-best-practices.md) for production patterns.

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

client = SecretClient(
    vault_url="https://<vault>.vault.azure.net/",
    credential=DefaultAzureCredential()
)
```

## Best Practices

- Use `DefaultAzureCredential` for **local development only**. In production, use `ManagedIdentityCredential` — see [auth-best-practices.md](../auth-best-practices.md)
- Use managed identity in Azure-hosted applications
- Enable soft-delete for recovery (enabled by default)
- Use RBAC over access policies for fine-grained control
- Rotate secrets regularly using versioning
- Use Key Vault references in App Service/Functions config
- Cache secrets appropriately to reduce API calls
- Use async clients for high-throughput scenarios
