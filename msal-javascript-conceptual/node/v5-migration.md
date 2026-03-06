---
title: Migrate from MSAL Node v3 to v5
description: Learn how to migrate your application from MSAL Node v3 to v5, including breaking changes and configuration updates.
author: Dickson-Mwendia
manager: Dougeby
ms.author: dmwendia
ms.date: 03/06/2026
ms.service: msal
ms.subservice: msal-node
ms.topic: how-to
ms.reviewer: cwerner, owenrichards, kengaderdus
#Customer intent: As a developer, I want to migrate my application from MSAL Node v3 to v5 so that I can use the latest features and security improvements.
---

# Migrate from MSAL Node v3 to v5

> [!NOTE]
> There is no MSAL Node v4 release. The package version was incremented from v3 directly to v5 to align `msal-node` versioning with the other MSAL.js libraries. No separate v4 feature set exists.

## Dropped support for Node.js 16 and 18

MSAL Node v5 no longer supports Node.js 16 or 18; you must use Node.js 20 or later.

## Removed `proxyUrl` and `customAgentOptions`

MSAL Node v5 no longer provides optional configuration for the HTTP client. The `proxyUrl` and `customAgentOptions` parameters have been removed from `NodeSystemOptions`.

```ts
// BEFORE (v3)
NodeSystemOptions = {
    loggerOptions?: LoggerOptions;
    networkClient?: INetworkModule;
    proxyUrl?: string;
    customAgentOptions?: http.AgentOptions | https.AgentOptions;
    disableInternalRetries?: boolean;
    protocolMode?: ProtocolMode;
};

// AFTER (v5)
NodeSystemOptions = {
    loggerOptions?: LoggerOptions;
    networkClient?: INetworkModule;
    disableInternalRetries?: boolean;
    protocolMode?: ProtocolMode;
};
```

Developers must now write their own custom HTTP client when proxy support is needed. See the [custom INetworkModule sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples/custom-INetworkModule-and-network-tracing) for implementation details.

## Configuration changes

### `protocolMode` moved to system config

The `protocolMode` parameter is no longer an auth config option and is instead a system config option.

```ts
// BEFORE (v3)
const msalConfig = {
    auth: {
        clientId: "your_client_id",
        authority: "https://login.live.com",
        protocolMode: "OIDC",
    },
};

// AFTER (v5)
const msalConfig = {
    auth: {
        clientId: "your_client_id",
        authority: "https://login.live.com",
    },
    system: {
        protocolMode: "OIDC",
    },
};
```

### Other removed parameters

- The `skipAuthorityMetadataCache` parameter has been removed. Applications no longer use the local metadata cache during authority initialization.
- The `encodeExtraQueryParams` parameter has been removed. All extra query parameters are automatically encoded.

## `fromNativeBroker` renamed to `fromPlatformBroker`

In the `AuthenticationResult` object, the `fromNativeBroker` field has been renamed to `fromPlatformBroker`.
