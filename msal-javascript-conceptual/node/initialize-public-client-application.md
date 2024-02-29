---
title: Initialize the public client application object in MSAL Node 
description: Learn how to initialize the PublicClientApplication object in MSAL Node and how to configure the authority.
author: EmLauber
manager: CelesteDG
ms.author: emilylauber

ms.date: 11/28/2023
ms.service: msal
ms.subservice: msal-node
ms.topic: how-to
ms.reviewer: dmwendia,cwerner, owenrichards, kengaderdus
#Customer intent: 
---

# Initialize the public client application object in MSAL Node 

In this article, you'll learn how to initialize the `PublicClientApplication` object in MSAL Node and how to configure the authority.

## Prerequisites

- [Node.js](https://nodejs.org/en/download/)
- An Azure account that has an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- An application registered in Microsoft Entra tenant. Refer to [Register an application with the Microsoft identity platform](/entra/identity-platform/quickstart-register-app).

## Initializing the PublicClientApplication object

In order to use MSAL Node, you need to instantiate a [PublicClientApplication](/javascript/api/@azure/msal-node/publicclientapplication) object. We support and strongly recommend the use of [PKCE](https://tools.ietf.org/html/rfc7636#section-6.2) (Proof Key for Code Exchange) for any PublicClientApplication. The usage pattern is demonstrated in the [PKCE Sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/master/samples/msal-node-samples/auth-code-pkce).

```javascript
import * as msal from "@azure/msal-node";

const clientConfig = {
    auth: {
        clientId: "your_client_id",
        authority: "your_authority",
    }
};
const pca = new msal.PublicClientApplication(clientConfig);
```

## Configuration Basics

[Configuration](/javascript/api/@azure/msal-node/configuration) options for node have `common` parameters and `specific` paremeters per authentication flow.

- `client_id` is mandatory to initialize a public client application
- `authority` defaults to `https://login.microsoftonline.com/common/` if the user does not set it during configuration

For more options on [Configuration](/javascript/api/@azure/msal-node/configuration) refer to [Configuration in MSAL Node](./configuration.md).

## Configure Authority

By default, MSAL is configured with the `common` tenant, which is used for multi-tenant applications and applications allowing personal accounts (not B2C).
```javascript
const msalConfig = {
    auth: {
        clientId: 'your_client_id',
        authority: 'https://login.microsoftonline.com/common/'
    }
};
```

If your application audience is a single tenant, you must provide an authority with your tenant id like below:

```javascript
const msalConfig = {
    auth: {
        clientId: 'your_client_id',
        authority: 'https://login.microsoftonline.com/{your_tenant_id}'
    }
};
```

If your application is using a separate OIDC-compliant authority like `"https://login.live.com"` or an IdentityServer, you will need to provide it in the `knownAuthorities` field and set your `protocolMode` to `"OIDC"`.

```javascript
const msalConfig = {
    auth: {
        clientId: 'your_client_id',
        authority: 'https://login.live.com',
        knownAuthorities: ["login.live.com"],
        protocolMode: "OIDC"
    }
};
```

## Next Steps

> [!div class="nextstepaction"]
> [Axquire tokens in MSAL Node](request.md)