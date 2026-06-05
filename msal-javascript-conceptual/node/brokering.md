---
title: "Using MSAL Node with the Native Token Broker (Windows)"
description: Learn how to acquire device-bound tokens from the native token broker on Windows using MSAL Node.
author: Dickson-Mwendia
manager: Dougeby
ms.author: dmwendia
ms.service: msal
ms.subservice: msal-node
ms.date: 06/05/2026
ms.topic: concept-article
ms.reviewer: kengaderdus
#Customer intent: As a developer, I want to learn how to acquire tokens from the native token broker on Windows.
---

# Using MSAL Node with the Native Token Broker (Windows)

MSAL Node supports acquiring tokens from the native token broker. When using the native broker, refresh tokens are bound to the device on which they are acquired and are not accessible by `msal-node` or the application. This provides a higher level of security that cannot be achieved by `msal-node` alone.

Broker support is available on the following platforms:

| Platform | Broker | Documentation |
|----------|--------|---------------|
| **Windows** | Web Account Manager (WAM) | This page |
| **macOS** | Microsoft Enterprise SSO plug-in (Company Portal) | [macOS broker](brokering-macos.md) |
| **Linux** | Microsoft Single Sign-on for Linux | [Linux broker](brokering-linux.md) |

## What is a broker

An authentication broker is a component that runs on a user's machine and manages authentication handshakes and token lifecycle for connected accounts. On Windows, this role is performed by Web Account Manager (WAM). Key benefits include:

- **Enhanced security.** Security improvements are delivered via OS or broker updates without requiring app code changes. Refresh tokens are device-bound and protected from exfiltration.
- **Feature support.** Access to rich OS capabilities such as Windows Hello, conditional access policies, and FIDO keys without extra scaffolding code.
- **System integration.** Applications plug into the built-in account picker, allowing users to quickly select an existing account instead of re-entering credentials.
- **Token protection.** The broker ensures refresh tokens are device-bound and [enables apps to acquire proof-of-possession access tokens](#proof-of-possession).

## Supported architectures

- Windows: x64, x86, ARM64

## Prerequisites

- Node.js 18 or later
- Install `@azure/msal-node-extensions` as a dependency
- Register the broker's redirect URI on your app registration (see [Redirect URI](#redirect-uri) below)

## Redirect URI

Register the following redirect URI under the **Mobile and desktop applications** platform in the Azure portal:

```text
ms-appx-web://Microsoft.AAD.BrokerPlugin/<your-client-id>
```

Replace `<your-client-id>` with your application's client ID.

## Enable the feature

Enabling token brokering requires just one configuration parameter:

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

> [!NOTE]
> `msal-node` will _not_ fall back to the non-brokered flow in the event of a failure. Only enable the broker flow in environments that support it to avoid unexpected failures.

A working sample can be found in the [auth-code-cli-brokered-app sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples/auth-code-cli-brokered-app).

## Window parenting

In order for auth prompts to display over the calling application and block further interaction, provide the application's window handle to the `acquireTokenInteractive` API.

For CLI apps, a best-effort attempt to find the window handle is made under the hood, but it is unreliable.

If you're using Electron, use the [`getNativeWindowHandle`](https://www.electronjs.org/docs/latest/api/browser-window#wingetnativewindowhandle) API and pass the result into `acquireTokenInteractive`:

```javascript
import { BrowserWindow } from "electron";

const win = new BrowserWindow();
const pca = new PublicClientApplication(msalConfig);

pca.acquireTokenInteractive({
    windowHandle: win.getNativeWindowHandle(),
});
```

## Proof of Possession

Access token proof of possession (PoP) is supported when acquiring tokens through the native broker. To request a PoP token, add the following properties to the request object provided to `acquireTokenInteractive` or `acquireTokenSilent`.

### AT PoP request parameters

| Name | Description | Required |
|---|---|---|
| `authenticationScheme` | Indicates whether MSAL should acquire a `Bearer` or `PoP` token. Default is `Bearer`. | **Required** |
| `resourceRequestMethod` | The all-caps name of the HTTP method of the request that will use the signed token (`GET`, `POST`, `PUT`, etc.) | **Required** |
| `resourceRequestUri` | The URL of the protected resource for which the access token is being issued | **Required** |
| `shrNonce` | A server-generated, signed timestamp that is Base64URL encoded as a string. This nonce is used to mitigate clock-skew and time-travel attacks meant to enable PoP token pre-generation. | *Optional* |

### Usage example

```javascript
import { PublicClientApplication, Configuration, AuthenticationScheme } from "@azure/msal-node";
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

const popTokenRequest = {
    scopes: ["User.Read"],
    authenticationScheme: AuthenticationScheme.POP,
    resourceRequestMethod: "POST",
    resourceRequestUri: "YOUR_RESOURCE_ENDPOINT",
    shrNonce: "NONCE_ACQUIRED_FROM_RESOURCE_SERVER",
};

pca.acquireTokenInteractive(popTokenRequest);
pca.acquireTokenSilent(popTokenRequest);
```

> [!NOTE]
> Access token proof-of-possession is only supported through the native broker flow and is not available in the non-brokered flow.

## Differences when using the broker to acquire tokens

There are a few things that may behave differently when acquiring tokens through the native broker:

- The `forceRefresh` parameter for `acquireTokenSilent` calls is not supported. You may receive a cached token from the broker regardless of what this flag is set to.
- If the broker needs to prompt the user for interaction, a system prompt will be opened. This is a UX change as authentication will not occur in a browser window.
- Access token proof-of-possession _is_ supported by the broker but _is not_ supported by the non-brokered flow.

