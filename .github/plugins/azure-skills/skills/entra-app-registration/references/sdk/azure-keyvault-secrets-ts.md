# Key Vault Secrets — TypeScript Library Quick Reference

> Condensed from **azure-keyvault-secrets-ts**. Full patterns (key rotation,
> cryptographic operations, backup/restore, wrap/unwrap)
> in the **azure-keyvault-secrets-ts** plugin skill if installed.

## Install

```bash
npm install @azure/keyvault-secrets @azure/identity
```

## Quickstart

```typescript
import { DefaultAzureCredential } from "@azure/identity";
import { SecretClient } from "@azure/keyvault-secrets";

const client = new SecretClient("https://<vault>.vault.azure.net", new DefaultAzureCredential());
```

## Best Practices

- Use `DefaultAzureCredential` for prototyping or **local development only**. In production, use `ManagedIdentityCredential` — see [auth-best-practices.md](../auth-best-practices.md)
- Enable soft-delete — required for production vaults
- Set expiration dates on both keys and secrets
- Use key rotation policies — automate key rotation
- Limit key operations — only grant needed operations (encrypt, sign, etc.)
- Browser not supported — these libraries are Node.js only
