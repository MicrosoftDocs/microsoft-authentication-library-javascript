---
title: Migrate from MSAL Browser v3 to v4
description: Learn how to migrate your browser application from MSAL Browser v3 to v4, including the loadExternalTokens async change and platform broker rename.
author: Dickson-Mwendia
manager: Dougeby
ms.author: dmwendia
ms.date: 03/15/2026
ms.service: msal
ms.subservice: msal-js
ms.topic: how-to
ms.reviewer: kengaderdus
#Customer intent: As a developer upgrading from MSAL Browser v3 to v4, I want a migration guide so I can update my app without breaking changes.
---

# Migrate from MSAL Browser v3 to v4

> [!NOTE]
> If you're migrating from v4 to v5, see [Migrate from MSAL Browser v4 to v5](v4-migration.md).

If you're new to MSAL, you should start with [Initialize applications](initialization.md).

If you're coming from MSAL v2, you should check [Migrate from MSAL Browser v2 to v3](v2-migration.md) first to migrate to MSAL v3 and then follow the next steps.

If you're coming from MSAL v3, you can follow this guide to update your code to use MSAL v4.

## API breaking changes

### loadExternalTokens API is now async

The [`loadExternalTokens` API](testing.md) is now async and returns a Promise. If you're using this API, you'll need to update your code to await the resolution of the promise before using its result.

```js
const msalTokenCache = myMSALObj.getTokenCache();

// v3
const authenticationResult = msalTokenCache.loadExternalTokens(
    silentRequest,
    serverResponse,
    loadTokenOptions
);

// v4 change this to:
const authenticationResult = await msalTokenCache.loadExternalTokens(
    silentRequest,
    serverResponse,
    loadTokenOptions
);
```

### allowNativeBroker renamed to allowPlatformBroker

The `allowNativeBroker` configuration parameter has been renamed to `allowPlatformBroker`. If you're using device-bound tokens, you'll need to update your configuration to continue using this feature. There aren't any other changes to behavior or default value. Read more about the platform broker in [Device-bound tokens](device-bound-tokens.md).

```js
// v3
const msalConfig = {
    auth: {
        clientId: "insert-clientId"
    },
    system: {
        allowNativeBroker: true
    }
};

// v4 change this to:
const msalConfig = {
    auth: {
        clientId: "insert-clientId"
    },
    system: {
        allowPlatformBroker: true
    }
};
```

## Behavioral breaking changes

The following changes don't require any code changes but are listed here as an FYI to changes in behavior of the library.

### LocalStorage encryption

Starting in v4, if you're using the `localStorage` cache location, auth artifacts are encrypted with [AES-GCM](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/encrypt#aes-gcm) using [HKDF](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/deriveKey#hkdf) to derive the key. The base key is stored in a session cookie titled `msal.cache.encryption`.

This cookie is automatically removed when the browser instance (not tab) is closed, thus making it impossible to decrypt any auth artifacts after the session has ended. These expired auth artifacts are removed the next time MSAL is initialized and the user might need to reauthenticate. The `localStorage` location still provides cross-tab cache persistence but won't persist across browser sessions.

> [!Important]
> The purpose of this encryption is to reduce the persistence of auth artifacts, **not** to provide additional security. If a bad actor gains access to browser storage, they'd also have access to the key or have the ability to request tokens on your behalf without the need for cache at all. It's your responsibility to ensure your application isn't vulnerable to XSS attacks.
