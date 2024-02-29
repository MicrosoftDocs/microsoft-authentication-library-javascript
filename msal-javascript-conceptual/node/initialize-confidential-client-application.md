---
title: How to initialize the confidential client application object in MSAL Node 
description: Learn how to initialize MSAL Node.
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

# How to initialize the confidential client application object in MSAL Node

This article shows you how to initialize the `ConfidentialClientApplication` object in MSAL Node. You'll learn how to use secrets and certificates securely, and how to configure the authority.

## Prerequisites

- [Node.js](https://nodejs.org/en/download/)
- An Azure account that has an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- An application registered in Microsoft Entra tenant. Refer to [Register an application with the Microsoft identity platform](/entra/identity-platform/quickstart-register-app).

## Initializing the `ConfidentialClientApplication` object

In order to use MSAL Node, you need to instantiate a [ConfidentialClient](/javascript/api/@azure/msal-node/confidentialclientapplication) object.

### Using secrets and certificates securely

Secrets should never be hardcoded. The dotenv npm package can be used to store secrets or certificates in a .env file (located in project's root directory) that should be included in *.gitignore* to prevent accidental uploads of the secrets.

Certificates can also be read-in from files via NodeJS's fs module. However, they should never be stored in the project's directory. Production apps should fetch certificates from [Azure KeyVault](https://azure.microsoft.com/products/key-vault), or other secure key vaults.

Please see [certificates and secrets](/entra/identity-platform/security-best-practices-for-app-registration#certificates-and-secrets) for more information.

See the MSAL sample: [auth-code-with-certs](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/master/samples/msal-node-samples/auth-code-with-certs)

```javascript
import * as msal from "@azure/msal-node";
import "dotenv/config"; // process.env now has the values defined in a .env file

const clientConfig = {
    auth: {
        clientId: "your_client_id",
        authority: "your_authority",
        clientSecret: process.env.clientSecret, // OR
        clientCertificate: {
            thumbprint: process.env.thumbprint,
            privateKey: process.env.privateKey
        }, // OR
        clientAssertion: "assertion"
    }
};
const pca = new msal.ConfidentialClientApplication(clientConfig);
```

Please refer to [Common issues when importing certificates](./certificate-credentials.md#common-issues).

## Configuration Basics

[Configuration](/javascript/api/@azure/msal-node/configuration) options for node have `common` parameters and `specific` paremeters per authentication flow.

- `clientId` is mandatory to initialize a public client application
- `authority` defaults to `https://login.microsoftonline.com/common/` if the user does not set it during configuration
- A Client credential is mandatory for confidential clients. Client credential can be a:
    - `clientSecret` is secret string generated set on the app registration.
    - `clientCertificate` is a certificate set on the app registration. The `thumbprint` is a X.509 SHA-1 thumbprint of the certificate, and the `privateKey` is the PEM encoded private key. `x5c` is the optional X.509 certificate chain used in [subject name/issuer auth scenarios](./sni.md).
    - `clientAssertion` is string that the application uses when requesting a token. The certificate used to sign the assertion should be set on the app registration. Assertion should be of type urn:ietf:params:oauth:client-assertion-type:jwt-bearer.

For more options on [Configuration](/javascript/api/@azure/msal-node/configuration) refer to [Configuration in MSAL Node](./configuration.md).

## Configure Authority

By default, MSAL is configured with the `common` tenant, which is used for multi-tenant applications and applications allowing personal accounts (not B2C).

```javascript
    authority: 'https://login.microsoftonline.com/common/'
```

If your application audience is a single tenant, you must provide an authority with your tenant ID like below:

```javascript
    authority: 'https://login.microsoftonline.com/{your_tenant_id}'
```

## Next Steps

> [!div class="nextstepaction"]
> [Axquire tokens in MSAL Node](request.md)
