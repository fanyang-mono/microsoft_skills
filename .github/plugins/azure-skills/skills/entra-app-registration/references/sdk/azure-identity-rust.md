# Authentication — Rust Library Quick Reference

> Condensed from **azure-identity-rust**. Full patterns (ClientSecret,
> ClientCertificate, WorkloadIdentity, AzurePipelines credentials)
> in the **azure-identity-rust** plugin skill if installed.

## Install

```bash
cargo add azure_identity
```

## Quickstart

> **Auth:** `DeveloperToolsCredential` is best for prototyping or local development. See [auth-best-practices.md](../auth-best-practices.md) for production patterns.

```rust
use azure_identity::DeveloperToolsCredential;

let credential = DeveloperToolsCredential::new(None)?;
```

## Best Practices

- Use `DeveloperToolsCredential` for local dev — automatically picks up Azure CLI
- Use `ManagedIdentityCredential` in production — no secrets to manage
- Clone credentials — credentials are Arc-wrapped and cheap to clone
- Reuse credential instances — same credential can be used with multiple clients
- Use tokio feature — `cargo add azure_identity --features tokio`
