---
title: Initialize the public client application object in MSAL Node 
description: Learn how to initialize the PublicClientApplication object in MSAL Node and how to configure the authority.
author: Dickson-Mwendia
manager: Dougebyms.author: dmwendia

ms.date: 05/21/2025
ms.service: msal
ms.subservice: msal-node
ms.topic: how-to
ms.reviewer: dmwendia,cwerner, owenrichards, kengaderdus
#Customer intent: 
---

# Initialize the public client application object in MSAL Node 

In this article, you'll learn how to initialize the `PublicClientApplication` object in MSAL Node and how to configure the authority.

## Prerequisites

Before initializing an application, you first need to [register it in the Microsoft Entra admin center](/entra/identity-platform/scenario-spa-app-registration), establishing a trust relationship between your application and the Microsoft identity platform.

After registering your app, you'll need some or all of the following values that can be found in the Microsoft Entra admin center.

| Value                   | Required | Description                                                                                                                                                                |
| :---------------------- | :------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Application (client) ID | Required | A GUID that uniquely identifies your application within the Microsoft identity platform.                                                                                   |
| Authority               | Optional | The identity provider URL (the _instance_) and the _sign-in audience_ for your application. The instance and sign-in audience, when concatenated, make up the _authority_. |
| Directory (tenant) ID   | Optional | Specify Directory (tenant) ID if you're building a line-of-business application solely for your organization, often referred to as a _single-tenant application_.          |
| Redirect URI            | Optional | If you're building a web app, the `redirectUri` specifies where the identity provider (the Microsoft identity platform) should return the security tokens it has issued.   |

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