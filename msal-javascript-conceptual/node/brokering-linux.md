---
title: "Using MSAL Node with the Linux Broker"
description: Learn how to use the Linux broker with MSAL Node to enable single sign-on, install dependencies, and configure secure brokered token acquisition.
author: Ugonnaak1
manager: iulico
ms.author: akaliugonna
ms.service: msal
ms.subservice: msal-node
ms.date: 06/05/2026
ms.topic: concept-article
ms.reviewer: dmwendia
#customer intent: As a developer building a Node.js app on Linux, I want to use the native broker to acquire tokens so that I can provide single sign-on and device-bound token protection.
---
# Using MSAL Node with the Linux Broker

Microsoft Authentication Library (MSAL) Node can use the Microsoft Single Sign-on for Linux broker to provide single sign-on (SSO) and secure token acquisition on supported Linux distributions. The broker manages authentication handshakes and token lifecycle, enabling users to benefit from integration with accounts known to the system.

This article explains what the Linux broker is, distributions it supports, and how to enable brokered token acquisition in a Node.js app.

For an overview of broker support across all platforms, see [Using MSAL Node with the Native Token Broker](brokering.md).

## What is the Linux broker

The Microsoft Single Sign-on for Linux is an authentication broker shipped independently of the Linux distribution. Install it with a package manager (`sudo apt install microsoft-identity-broker` or `sudo dnf install microsoft-identity-broker`). It's also bundled as a dependency of Microsoft applications such as [Company Portal](/mem/intune-service/user-help/enroll-device-linux).

Key benefits include:

- **Single sign-on.** Simplifies how users authenticate with Microsoft Entra ID and protects refresh tokens from exfiltration and misuse.
- **Enhanced security.** Security improvements are delivered via broker updates without requiring app code changes.
- **Token protection.** The broker ensures refresh tokens are device-bound.
- **System integration.** Applications plug into the built-in account picker, allowing users to quickly select an existing account.

## Supported distributions

- Ubuntu 22.04 / 24.04 (x64)
- Red Hat Enterprise Linux (RHEL) 8 / 9 / 10 (x64)

## Prerequisites

- Node.js 18 or later
- Install `@azure/msal-node-extensions` as a dependency
- `microsoft-identity-broker` must be installed on the device
- Register the broker redirect URI on your app registration (see [Redirect URI](#redirect-uri) below)
- Install the required system dependencies (see [System dependencies](#system-dependencies) below)

## Redirect URI

For Linux broker flows, register the following redirect URI under the **Mobile and desktop applications** platform in the Azure portal:

```text
https://login.microsoftonline.com/common/oauth2/nativeclient
```

## System dependencies

# [Ubuntu](#tab/ubuntu)

### Ubuntu 22.04/24.04

```bash
sudo apt install libsecret-1-0 libdbus-1-3
```

- **libsecret** — Securely stores and retrieves authentication tokens via the system keyring (GNOME Keyring or KDE Wallet).
- **dbus** — Inter-process communication with the authentication broker. A D-Bus session bus must be running (standard in desktop environments; headless servers may need `dbus-run-session` or equivalent).

```bash
sudo apt install libwebkit2gtk-4.1-0 libgtk-3-0
```

- **WebKitGTK** — Provides the embedded browser for the interactive sign-in UI.
- **GTK** — The underlying UI toolkit required by WebKitGTK.

# [RHEL](#tab/rhel)

### RHEL 8

```bash
sudo dnf install libsecret dbus-libs openssl3-libs
```

- **libsecret** — Securely stores and retrieves authentication tokens via the system keyring (GNOME Keyring or KDE Wallet).
- **dbus-libs** — Inter-process communication with the authentication broker. A D-Bus session bus must be running (standard in desktop environments; headless servers may need `dbus-run-session` or equivalent).
- **openssl3-libs** — The native binary is linked against OpenSSL 3 (`libcrypto.so.3`). RHEL 8 ships with OpenSSL 1.1.1 by default, so this package must be explicitly installed. Many RHEL 8 systems already have it as a dependency of other software.

```bash
sudo dnf install webkit2gtk3 gtk3
```

### RHEL 9

```bash
sudo dnf install libsecret dbus-libs
```

No extra OpenSSL packages needed — OpenSSL 3 is the system default.

```bash
sudo dnf install webkit2gtk3 gtk3
```

### RHEL 10

```bash
sudo dnf install libsecret dbus-libs
```

No extra OpenSSL packages needed — OpenSSL 3 is the system default.

```bash
sudo dnf install webkitgtk6.0 gtk4
```

---

## Enable the feature

Enabling the Linux broker uses the same configuration as Windows and macOS. Pass a `NativeBrokerPlugin` instance in the broker configuration:

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
> `msal-node` doesn't fall back to a browser-based flow if the broker is unavailable. Only enable brokered authentication on supported Linux devices to avoid unexpected failures.

## Acquiring tokens

### Interactive token acquisition

Use `acquireTokenInteractive` to request a token through the Linux broker:

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

The authentication broker handles refresh and access token caching. Tokens acquired through the broker are managed by the broker itself and are device-bound. You do not need to set up custom caching when using the broker.

## Differences when using the Linux broker

- When the broker needs to prompt for interaction, a native sign-in dialog (WebKitGTK-based) appears. This changes the user experience (UX) compared to browser-based authentication.
- The `forceRefresh` parameter for `acquireTokenSilent` is not supported. You may receive a cached token from the broker regardless of this flag.
- A D-Bus session bus must be running for broker communication to work.

## Limitations

- Azure AD B2C and Active Directory Federation Services (AD FS) authorities are not supported via the Linux broker.
- Third-party identity providers (IDPs) are not supported.
- `microsoft-identity-broker` must be installed on the device for the broker to function.
- `msal-node` does not fall back to a browser if the broker is unavailable. Only enable the broker in environments that support it.
- Access token proof-of-possession (PoP) is not currently supported through the Linux broker.
