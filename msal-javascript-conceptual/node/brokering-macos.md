---
title: "Using MSAL Node with the macOS Broker"
description: Learn how to acquire device-bound tokens from the native authentication broker on macOS using MSAL Node.
author: Ugonnaak1
manager: iulico
ms.author: akaliugonna
ms.service: msal
ms.subservice: msal-node
ms.date: 06/05/2026
ms.topic: concept-article
ms.reviewer: dmwendia
#Customer intent: As a developer, I want to learn how to acquire tokens from the native broker on macOS.
---
# Using MSAL Node with the macOS Broker

MSAL Node can integrate with the macOS authentication broker to provide single sign-on (SSO) and secure token acquisition by leveraging accounts known to the operating system. This allows users to sign in using an existing Microsoft Entra ID account with fewer prompts and improved security.

For an overview of broker support across all platforms, see [Using MSAL Node with the Native Token Broker](brokering.md).

## What is the macOS broker

On macOS, the authentication broker is provided by the [Microsoft Enterprise SSO plug-in for Apple devices](/en-us/entra/identity-platform/apple-sso-plugin), which ships with the **Company Portal** app. The broker manages authentication handshakes and token lifecycle for connected accounts. Key benefits include:

- **Enhanced security.** Security improvements are delivered via broker updates without requiring app code changes. Refresh tokens are device-bound and protected from exfiltration.
- **System integration.** Users can reuse existing signed-in accounts from Company Portal, reducing credential re-entry.
- **Token protection.** The broker ensures refresh tokens are bound to the device context.

## Supported platforms and architectures

| Component               | Supported                             |
| ----------------------- | ------------------------------------- |
| **Architecture**  | ARM64 (Apple Silicon) and x64 (Intel) |
| **macOS version** | macOS 10.15 (Catalina) and later      |

> [!TIP]
> We recommend updating to the latest macOS version to ensure compatibility with the newest security features and broker capabilities.

## Prerequisites

- Node.js 18 or later
- Install `@azure/msal-node-extensions` as a dependency
- The device must be enrolled via [Company Portal](/en-us/intune/intune-service/user-help/enroll-your-device-in-intune-macos-cp). After enrollment, verify that other Microsoft apps can sign in via the SSO extension (for example, you can sign in to Word via Company Portal).
- Register the broker redirect URI on your app registration (see [Redirect URI](#redirect-uri) below)

## Redirect URI

For macOS broker flows, you must register a platform-specific redirect URI under the **Mobile and desktop applications** platform in the Azure portal.

**For unsigned applications (scripts, CLI tools):**

```text
msauth.com.msauth.unsignedapp://auth
```

**For signed/bundled applications:**

```text
msauth.<your-bundle-id>://auth
```

Replace `<your-bundle-id>` with your application's Apple bundle identifier (e.g., `msauth.com.example.myapp://auth`).

> [!IMPORTANT]
> The broker redirect URI should **only** be used for broker flows. If your application also uses browser-based authentication flows, use a separate redirect URI for those. Providing the broker redirect URI to a browser-based flow will result in an error.

## Enable the feature

Enabling the macOS broker requires the same configuration as Windows — just one parameter:

```javascript
import { PublicClientApplication, Configuration } from "@azure/msal-node";
import { NativeBrokerPlugin } from "@azure/msal-node-extensions";

const msalConfig: Configuration = {
    auth: {
        clientId: "your-client-id",
    },
    broker: {
        nativeBrokerPlugin: new NativeBrokerPlugin(),
    },
};

const pca = new PublicClientApplication(msalConfig);
```

## Acquiring tokens

### Interactive token acquisition

```javascript
const tokenRequest = {
    scopes: ["User.Read"],
};

const result = await pca.acquireTokenInteractive(tokenRequest);
console.log("Access token:", result.accessToken);
```

### Silent token acquisition

After an initial interactive sign-in, subsequent token requests can be made silently:

```javascript
const accounts = await pca.getAllAccounts();

if (accounts.length > 0) {
    const silentRequest = {
        scopes: ["User.Read"],
        account: accounts[0],
    };

    const result = await pca.acquireTokenSilent(silentRequest);
    console.log("Access token (silent):", result.accessToken);
}
```

## Token caching

The authentication broker handles refresh and access token caching. Tokens acquired through the broker are managed by the broker itself and are device-bound.

## Differences when using the macOS broker

- When the broker needs to prompt for interaction, a native macOS authentication dialog will appear. This is a UX change compared to browser-based authentication.
- The `forceRefresh` parameter for `acquireTokenSilent` is not supported. You may receive a cached token from the broker regardless of this flag.
- Access token proof-of-possession (PoP) is supported through the broker.

## Limitations

- Azure AD B2C and Active Directory Federation Services (ADFS) authorities are not supported via the macOS broker.
- Third-party identity providers (IDPs) are not supported.
- Company Portal must be installed and the device must be enrolled for the broker to function.
- `msal-node` does not fall back to a browser if the broker is unavailable. Only enable the broker in environments that support it.
