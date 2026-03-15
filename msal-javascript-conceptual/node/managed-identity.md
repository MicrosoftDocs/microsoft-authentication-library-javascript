---
title: Use managed identity with MSAL Node
description: Learn how to use managed identities in Azure with MSAL Node to acquire tokens without managing secrets manually.
author: Dickson-Mwendia
manager: Dougeby
ms.author: dmwendia
ms.date: 03/06/2026
ms.service: msal
ms.subservice: msal-node
ms.topic: how-to
ms.reviewer: kengaderdus
#Customer intent: As a developer, I want to use managed identity with MSAL Node so that I can acquire tokens without manually managing secrets and credentials.
---

# Use managed identity with MSAL Node

A common challenge for developers is the management of secrets, credentials, certificates, and keys used to secure communication between services. [Managed identities](/entra/identity/managed-identities-azure-resources/overview) in Azure eliminate the need for developers to handle these credentials manually. MSAL Node supports acquiring tokens through the managed identity service when used with applications running inside Azure infrastructure, such as:

- [Azure App Service](https://azure.microsoft.com/products/app-service/) (API version `2019-08-01` and above)
- [Azure VMs](https://azure.microsoft.com/free/virtual-machines/)
- [Azure Arc](/azure/azure-arc/overview)
- [Azure Cloud Shell](/azure/cloud-shell/overview)
- [Azure Service Fabric](/azure/service-fabric/service-fabric-overview)
- [Azure Machine Learning](https://azure.microsoft.com/products/machine-learning)

For a complete list, refer to [Azure services that can use managed identities to access other services](/entra/identity/managed-identities-azure-resources/managed-identities-status).

> [!NOTE]
> Browser-based MSAL libraries do not offer managed identity because the browser is not hosted on Azure.

## Which SDK to use — Azure SDK or MSAL?

Both MSAL Node and [Azure SDK](/javascript/api/overview/azure/identity-readme) allow tokens to be acquired via managed identity. Internally, Azure SDK uses MSAL Node, and it provides a higher-level API via its `DefaultAzureCredential` and `ManagedIdentityCredential` abstractions.

If your application already uses one of the SDKs, continue using the same SDK. Use Azure SDK if you are writing a new application and plan to call other Azure resources, as this SDK provides a better developer experience by allowing the app to run on private developer machines where managed identity doesn't exist. Consider using MSAL if you need to call other downstream web APIs like Microsoft Graph or your own web API.

To get started and see Azure Managed Identity in action, you can use one of [the MSAL Node managed identity samples](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples/Managed-Identity).

## How to use managed identities

There are two types of managed identities available to developers - **system-assigned** and **user-assigned**. You can learn more about the differences in the [Managed identity types](/entra/identity/managed-identities-azure-resources/overview#managed-identity-types) article. MSAL Node supports acquiring tokens with both.

Before you can use managed identities from MSAL Node, you must enable them for the resources they want to use through Azure CLI or the Azure portal.

For both user-assigned and system-assigned identities, developers can use the `ManagedIdentityApplication` class.

### System-assigned managed identities

For system-assigned managed identities, the developer doesn't need to pass any additional information when creating an instance of `ManagedIdentityApplication`, as it will automatically infer the relevant metadata about the assigned identity.

[acquireToken](/javascript/api/@azure/msal-node/managedidentityapplication) is called with the resource to acquire a token for, such as `https://management.azure.com`.

```typescript
import {
    ManagedIdentityApplication,
    ManagedIdentityConfiguration,
    ManagedIdentityRequestParams,
    NodeSystemOptions,
    LoggerOptions,
    LogLevel,
    AuthenticationResult
} from "@azure/msal-node";

const config: ManagedIdentityConfiguration = {
    system: {
        loggerOptions: {
            logLevel: LogLevel.Verbose,
        } as LoggerOptions,
    } as NodeSystemOptions,
};

const systemAssignedManagedIdentityApplication: ManagedIdentityApplication =
    new ManagedIdentityApplication(config);

const managedIdentityRequestParams: ManagedIdentityRequestParams = {
    resource: "https://management.azure.com",
};

const response: AuthenticationResult =
    await systemAssignedManagedIdentityApplication.acquireToken(
        managedIdentityRequestParams
    );
console.log(response);
```

### User-assigned managed identities

For user-assigned managed identities, the developer needs to pass either the client ID, full resource identifier, or the object ID of the managed identity when creating `ManagedIdentityApplication`.

Like in the case for system-assigned managed identities, `acquireToken` is called with the resource to acquire a token for.

```typescript
import {
    ManagedIdentityApplication,
    ManagedIdentityConfiguration,
    ManagedIdentityIdParams,
    ManagedIdentityRequestParams,
    NodeSystemOptions,
    LoggerOptions,
    LogLevel,
    AuthenticationResult
} from "@azure/msal-node";

const managedIdentityIdParams: ManagedIdentityIdParams = {
    userAssignedClientId: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
};

const config: ManagedIdentityConfiguration = {
    managedIdentityIdParams,
    system: {
        loggerOptions: {
            logLevel: LogLevel.Verbose,
        } as LoggerOptions,
    } as NodeSystemOptions,
};

const userAssignedManagedIdentityApplication: ManagedIdentityApplication =
    new ManagedIdentityApplication(config);

const managedIdentityRequestParams: ManagedIdentityRequestParams = {
    resource: "https://management.azure.com",
};

const response: AuthenticationResult =
    await userAssignedManagedIdentityApplication.acquireToken(
        managedIdentityRequestParams
    );
console.log(response);
```

## Caching

MSAL Node caches tokens from managed identity in memory. There is no eviction, but memory is not a concern because a limited number of managed identities can be defined. Cache extensibility is not supported in this scenario because tokens should not be shared between machines.

## Troubleshooting commmon errors

For failed requests, the error response contains a correlation ID that can be used for further diagnostics and log analysis. Keep in mind that the correlation IDs generated in MSAL or passed into MSAL are different from the one returned in server error responses, as MSAL can't pass the correlation ID to managed identity token acquisition endpoints.

#### `ManagedIdentityError` — Error Code: `invalid_resource`

**Error Message**: The supplied resource is an invalid URL.

This exception might mean that the resource you are trying to acquire a token for is either not supported or is provided using the wrong resource ID format. Examples of correct resource ID formats include `https://management.azure.com/.default`, `https://management.azure.com`, and `https://graph.microsoft.com`.
