---
title: Use MCP flows with MSAL Node
description: Learn how to enable Model Context Protocol (MCP) flows in MSAL Node to acquire resource-scoped tokens for MCP applications.
author: Dickson-Mwendia
manager: Dougeby
ms.author: dmwendia
ms.date: 03/15/2026
ms.service: msal
ms.subservice: msal-node
ms.topic: how-to
ms.reviewer: kengaderdus
#Customer intent: As a developer building MCP applications, I want to use MSAL Node to acquire resource-scoped tokens so that my MCP server can authenticate against multiple resources.
---

# Use MCP flows with MSAL Node

When building [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) applications, you can configure MSAL Node to enforce resource-scoped token acquisition and caching. When MCP mode is enabled, MSAL requires every token request to include a `resource` parameter and caches access tokens keyed by that resource.

MCP flows are available for public client applications only.

## Enable MCP mode

Set `isMcp: true` in the `auth` configuration when creating your `PublicClientApplication`:

```javascript
const config = {
    auth: {
        clientId: "your-client-id",
        authority: "https://login.microsoftonline.com/common",
        isMcp: true,
    },
};

const pca = new msal.PublicClientApplication(config);
```

## Include the resource parameter

When `isMcp` is `true`, every token request **must** include a `resource` parameter. Omitting it throws a `resource_parameter_required` error.

```javascript
const tokenRequest = {
    scopes: ["User.Read"],
    redirectUri: "http://localhost:3000/redirect",
    resource: "https://example.microsoft.com",
    code: authorizationCode,
};

const response = await pca.acquireTokenByCode(tokenRequest);
```

> [!IMPORTANT]
> Set the `resource` parameter directly on the request object. Do **not** pass it via `extraQueryParameters` at the same time as the `resource` property — doing so throws a `misplaced_resource_parameter` error.

The following example shows the correct and incorrect ways to set the `resource` parameter:

```javascript
// Correct — resource on the request object
const request = {
    scopes: ["User.Read"],
    resource: "https://example.microsoft.com",
};

// Wrong — resource in both locations
const request = {
    scopes: ["User.Read"],
    resource: "https://example.microsoft.com",
    extraQueryParameters: { resource: "https://example.microsoft.com" },
};
```

## Resource-scoped caching

When `isMcp` is enabled, access tokens are cached with their associated resource. This affects silent token acquisition:

- **Cache hit**: If a cached access token exists for the same scopes **and** resource, it's returned from cache.
- **Cache miss**: If the requested resource doesn't match any cached token, MSAL falls back to the network to acquire a new token for the requested resource.

```javascript
const msalTokenCache = pca.getTokenCache();
const accounts = await msalTokenCache.getAllAccounts();

// First request — acquires token from network
const token1 = await pca.acquireTokenSilent({
    scopes: ["User.Read"],
    resource: "https://resource-a.microsoft.com",
    account: accounts[0],
});

// Same resource — returns cached token
const token2 = await pca.acquireTokenSilent({
    scopes: ["User.Read"],
    resource: "https://resource-a.microsoft.com",
    account: accounts[0],
});

// Different resource — falls back to network
const token3 = await pca.acquireTokenSilent({
    scopes: ["User.Read"],
    resource: "https://resource-b.microsoft.com",
    account: accounts[0],
});
```

## Error handling

Two errors are specific to MCP flows:

| Error code | Description |
|---|---|
| `resource_parameter_required` | `isMcp` is `true` but the request doesn't include a `resource` parameter. |
| `misplaced_resource_parameter` | A `resource` was found both in the `resource` property and in `extraQueryParameters`. Use only one. |

Both errors are thrown as `ClientAuthError`. For more information, see [MSAL Node FAQs](faq.md).

## Samples

- [MCP Flows Sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples/mcp-flows) -Express app demonstrating MCP with authorization code and silent flows.
