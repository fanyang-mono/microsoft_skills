# Azure Authentication Best Practices

> Source: [Microsoft — Passwordless connections for Azure services](https://learn.microsoft.com/azure/developer/intro/passwordless-overview) and [Azure Identity client libraries](https://learn.microsoft.com/dotnet/azure/sdk/authentication/).

## Golden Rule

Use **managed identities** and **Azure RBAC** in production. Reserve `DefaultAzureCredential` for **local development only**.

## Authentication by Environment

| Environment                   | Recommended Credential                                                                            | Why                                         |
|-------------------------------|---------------------------------------------------------------------------------------------------|---------------------------------------------|
| **Production (Azure-hosted)** | `ManagedIdentityCredential` (system- or user-assigned)                                            | No secrets to manage; auto-rotated by Azure |
| **Production (on-premises)**  | `ClientCertificateCredential` (`CertificateCredential` in Python) or `WorkloadIdentityCredential` | Deterministic; no fallback chain overhead   |
| **CI/CD pipelines**           | `AzurePipelinesCredential`                                                                        | Scoped to pipeline identity                 |
| **Local development**         | `DefaultAzureCredential`                                                                          | Chains dev tool credentials for convenience |

## Why Not DefaultAzureCredential in Production?

1. **Unpredictable fallback chain** — walks through multiple credential types, adding latency and making failures harder to diagnose.
2. **Broad surface area** — checks environment variables, CLI tokens, and other sources that should not exist in production.
3. **Non-deterministic** — which credential actually authenticates depends on the environment, making behavior inconsistent across deployments.
4. **Performance** — each failed credential attempt adds network round-trips before falling back to the next.

## Environment-Aware Pattern

Detect the runtime environment and select the appropriate credential. The key principle is to use `DefaultAzureCredential` only when running locally, and a specific credential in production. Set the `AZURE_TOKEN_CREDENTIALS` environment variable to `"dev"` or a specific developer tool credential (e.g., `"AzureCli"`, `"AzurePowerShell"`, `"VisualStudioCode"`) to skip unnecessary credential checks and speed up authentication in your local environment.

See the language-specific overviews for the full `DefaultAzureCredential` credential chain and its configuration options:

- [.NET](https://aka.ms/azsdk/net/identity/credential-chains#defaultazurecredential-overview)
- [Go](https://aka.ms/azsdk/go/identity/credential-chains#defaultazurecredential-overview)
- [Java](https://aka.ms/azsdk/java/identity/credential-chains#defaultazurecredential-overview)
- [JavaScript](https://aka.ms/azsdk/js/identity/credential-chains#use-defaultazurecredential-for-flexibility)
- [Python](https://aka.ms/azsdk/python/identity/credential-chains#defaultazurecredential-overview)

> **Tip:** Azure Functions sets `AZURE_FUNCTIONS_ENVIRONMENT` to `"Development"` when running locally. For App Service or containers, use any environment variable you control (e.g. `NODE_ENV`, `ASPNETCORE_ENVIRONMENT`).

### .NET

```csharp
using Azure.Identity;

var credential = Environment.GetEnvironmentVariable("AZURE_FUNCTIONS_ENVIRONMENT") == "Development"
    ? new DefaultAzureCredential(DefaultAzureCredential.DefaultEnvironmentVariableName)   // local dev — uses dev tool credentials
    : new ManagedIdentityCredential(ManagedIdentityId.SystemAssigned);                    // production — deterministic, no fallback chain
// For user-assigned managed identity:
// new ManagedIdentityCredential(ManagedIdentityId.FromUserAssignedClientId("<client-id>"))
```

### Go

```go
import (
    "os"

    "github.com/Azure/azure-sdk-for-go/sdk/azcore"
    "github.com/Azure/azure-sdk-for-go/sdk/azidentity"
)

func getCredential() (azcore.TokenCredential, error) {
    if os.Getenv("AZURE_FUNCTIONS_ENVIRONMENT") == "Development" {
        opts := azidentity.DefaultAzureCredentialOptions{RequireAzureTokenCredentials: true}
        return azidentity.NewDefaultAzureCredential(&opts)   // local dev — uses dev tool credentials
    }
    return azidentity.NewManagedIdentityCredential(nil)      // production — deterministic, no fallback chain
    // For user-assigned managed identity:
    // azidentity.NewManagedIdentityCredential(&azidentity.ManagedIdentityCredentialOptions{
    //     ID: azidentity.ClientID("<client-id>"),
    // })
}
```

### Java

```java
import com.azure.identity.DefaultAzureCredentialBuilder;
import com.azure.identity.ManagedIdentityCredentialBuilder;

var credential = "Development".equals(System.getenv("AZURE_FUNCTIONS_ENVIRONMENT"))
    ? new DefaultAzureCredentialBuilder()
        .requireEnvVars(AzureIdentityEnvVars.AZURE_TOKEN_CREDENTIALS)
        .build()                                      // local dev — uses dev tool credentials
    : new ManagedIdentityCredentialBuilder().build(); // production — deterministic, no fallback chain
// For user-assigned managed identity:
// new ManagedIdentityCredentialBuilder().clientId("<client-id>").build()
```

### Python

```python
import os
from azure.identity import DefaultAzureCredential, ManagedIdentityCredential

credential = (
    DefaultAzureCredential(require_envvar=True)           # local dev — uses dev tool credentials
    if os.getenv("AZURE_FUNCTIONS_ENVIRONMENT") == "Development"
    else ManagedIdentityCredential()                      # production — deterministic, no fallback chain
)
# For user-assigned managed identity:
# ManagedIdentityCredential(client_id="<client-id>")
```

### TypeScript / JavaScript

```typescript
import { DefaultAzureCredential, ManagedIdentityCredential } from "@azure/identity";

const credential = process.env.NODE_ENV === "development"
  ? new DefaultAzureCredential({ 
        requiredEnvVars: [ "AZURE_TOKEN_CREDENTIALS" ]
    })                                 // local dev — uses dev tool credentials
  : new ManagedIdentityCredential();   // production — deterministic, no fallback chain
// For user-assigned managed identity:
// new ManagedIdentityCredential({ clientId: "<client-id>"})
```

## Security Checklist

- [ ] Use managed identity for all Azure-hosted apps
- [ ] Never hardcode credentials, connection strings, or keys
- [ ] Apply least-privilege RBAC roles at the narrowest scope
- [ ] Use `ManagedIdentityCredential` (not `DefaultAzureCredential`) in production
- [ ] Store any required secrets in Azure Key Vault
- [ ] Rotate secrets and certificates on a schedule
- [ ] Set `AZURE_TOKEN_CREDENTIALS` in local dev environments to restrict the `DefaultAzureCredential` credential chain
- [ ] Enable Azure SDK diagnostic logging to troubleshoot authentication failures
- [ ] Enable Microsoft Defender for Cloud on production resources

## Further Reading

- [Passwordless connections overview](https://learn.microsoft.com/azure/developer/intro/passwordless-overview)
- [Managed identities overview](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/overview)
- [Azure RBAC overview](https://learn.microsoft.com/azure/role-based-access-control/overview)
- [Azure Identity library for .NET](https://learn.microsoft.com/dotnet/api/overview/azure/identity-readme)
- [Azure Identity library for Go](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/azidentity#section-readme)
- [Azure Identity library for Java](https://learn.microsoft.com/java/api/overview/azure/identity-readme)
- [Azure Identity library for JavaScript](https://learn.microsoft.com/javascript/api/overview/azure/identity-readme)
- [Azure Identity library for Python](https://learn.microsoft.com/python/api/overview/azure/identity-readme)
