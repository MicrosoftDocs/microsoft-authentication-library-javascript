---
title: Token caching in MSAL Node
description: Learn how to effectively cache tokens in MSAL Node, and use client secrets securely,
author: EmLauber
manager: CelesteDG
ms.author: emilylauber

ms.date: 11/28/2023
ms.service: msal
ms.subservice: msal-node
ms.topic: conceptual
ms.reviewer: dmwendia, cwerner, owenrichards, kengaderdus
#Customer intent: 
---

# Token caching in MSAL Node

When MSAL Node acquires a token, it caches it in memory for future usage. MSAL Node manages the token lifetime and refreshing for you. APIs like `acquireTokenSilent()` retrieves access tokens from the cache for a given account:

MSAL does not expose refresh tokens for security reasons. Refer to the [FAQ](./faq.md#how-do-i-get-the-refresh-token) for how to get the refresh tokens.

## Use client secrets securely

Client secrets should never be hardcoded. The dotenv npm package can be used to store secrets in a *.env* file (located in project's root directory) that should be included in *.gitignore* to prevent accidental uploads of the secrets.

```javascript
const msal = require('@azure/msal-node');
require('dotenv').config(); // process.env now has the values defined in a .env file

// Create msal application object
const cca = new msal.ConfidentialClientApplication({
    auth: {
        clientId: "Enter_the_Application_Id_Here", // e.g. "b1b60dca-c49d-496e-9851-xxxxxxxxxxxx" (guid)
        authority: "https://login.microsoftonline.com/Enter_the_Tenant_Info_Here", // e.g. "common" or your tenantId (guid)
        clientSecret: process.env.clientSecret // obtained during app registration
    }
});

/**
* acquireToken* APIs return an account object containing the "homeAccountId"
* you should keep a record of this in your app and use it later on when calling acquireTokenSilent
*/
const someUserHomeAccountId = "Enter_User_Home_Account_Id";

const msalTokenCache = cca.getTokenCache();
const account = await msalTokenCache.getAccountByHomeId(someUserHomeAccountId);

const silentTokenRequest = {
    account: account,
    scopes: ["User.Read"],
};

cca.acquireTokenSilent(silentTokenRequest).then((response) => {
    // do something with response
}).catch((error) => {
    // catch and handle errors
});
```

In production, you'd most likely want to serialize and persist the token cache. Depending on the type of application, you can:

* Desktop apps, console apps (public clients apps (PCA)):
  * Use [MSAL Node Extensions](./extensions.md), which provides persistence and encryption at rest solutions on Windows, Linux and Mac OS
* Web apps, web APIs, daemon apps (confidential client apps (CCA)):
  * MSAL's in-memory token cache does not scale for production. Use the [distributed token caching](#performance-and-security) pattern to persist the cache in your choice of storage environment (Redis, MongoDB, SQL databases etc. -keep in mind that you can use these in tandem *e.g.* a Redis-like memory cache as a first layer of persistence, and a SQL database as a second, more stable persistence layer)

## In-memory cache

MSAL maintains an in-memory cache. The in-memory cache is representative of the application cache state. The lifetime of in-memory cache is the same as the MSAL application object. If the process using MSAL restarts, the cache is erased when the process lifecycle finishes. If the in-memory cache is empty and there is no persistent cache to restore the cache from, users will have to re-authenticate. When this happens, if the user still has an active session with Microsoft Entra ID, they might re-authenticate without any prompts, however this still degrades the user experience. Service-to-service scenarios (i.e. client credentials flow, on-behalf-of flow) also suffer because getting a token from Microsoft Entra ID involves HTTP requests, and is much slower than getting a token from cache.

Note that the in-memory cache is not scalable for server-side applications and performance will degrade after holding a few 100 tokens in cache. For web app and web API scenarios, this approximates to serving a few 100 users. For daemon app scenarios using client credentials grant to call other apps, this means a few 100 tenants. See [performance](#performance-and-security) below for more.

> :warning: We recommend **persisting** the cache with **encryption** for all production applications both for security and desired cache longevity. If you choose not to persist the cache, the [TokenCache](/javascript/api/@azure/msal-node/tokencache) interface is still available to access the cached entities.

## Persistent cache

MSAL Node fires events when the in-memory cache is accessed and apps can choose whether to persist the cache (see: [TokenCacheContext](https://azuread.github.io/microsoft-authentication-library-for-js/ref/classes/_azure_msal_node.TokenCacheContext.html)) (e.g. to a file, a SQL database and etc.). This constitutes two actions:

1. Load the cache from persistence to MSAL's memory before accessing the cache
2. If the in-memory cache has changed since last access, save the cache back to persistence

For persisting the cache, MSAL accepts a custom cache plugin in [configuration](./configuration.md). This plugin should implement the [ICachePlugin](https://azuread.github.io/microsoft-authentication-library-for-js/ref/interfaces/_azure_msal_node.ICachePlugin.html) interface:

```typescript
interface ICachePlugin {
    beforeCacheAccess: (tokenCacheContext: TokenCacheContext) => Promise<void>;
    afterCacheAccess: (tokenCacheContext: TokenCacheContext) => Promise<void>;
}
```

A basic implementation of `ICachePlugin` interface might look as follows (see also [performance and security](#performance-and-security) if you are building a server-side app):

```typescript
class MyCachePlugin implements ICachePlugin {
    private client: ICacheClient;

    constructor(client: ICacheClient) {
        this.client = client; // client object to access the persistent cache
    }

    public async beforeCacheAccess(cacheContext: TokenCacheContext): Promise<void> {
        const cacheData = await this.client.get(); // get the cache from persistence
        cacheContext.tokenCache.deserialize(cacheData); // deserialize it to in-memory cache
    }

    public async afterCacheAccess(cacheContext: TokenCacheContext): Promise<void> {
        if (cacheContext.cacheHasChanged) {
            await this.client.set(cacheContext.tokenCache.serialize()); // deserialize in-memory cache to persistence
        }
    }
}
```

* If you are developing a public client app, [MSAL Node Extensions](./extensions.md) handles this for you.
* If you are developing a confidential client app, you should persist the cache via a separate service, since a single, *per-server* cache instance isn't suitable for a cloud environment with many servers and app instances.

We strongly recommend to encrypt the token cache when persisting it on disk. For public client apps, this is offered out-of-box with MSAL Node Extensions. For confidential clients however, you are responsible for devising an appropriate encryption solution.

## Performance and security

On public client apps, MSAL Node Extensions ensures performance and security for you.

On confidential client apps that handle users (web apps that sign in users and call web APIs, and web APIs calling downstream web APIs), there can be many users active concurrently for a given application. Our recommendation is to serialize one cache blob (see [CacheRecord](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/src/cache/entities/CacheRecord.ts)) per user. This would help with scaling the cache across a distributed system. Use a key for partitioning the cache (*i.e.* **partition key**), such as:

* For web apps: `<userObjectId>.<tenantId>` (i.e. `homeAccountId`)
* For multi-tenant daemon apps using client credentials grant: `<clientId>.<tenantId>`
* For web APIs calling other web APIs using OBO: hash of the incoming access token (i.e. `oboAssertion`) -the token which will subsequently be exchanged for an OBO token

> :warning: Please make sure to see [performance](./performance.md) for more information on how to monitor usage and avoid poor performance.

### Web apps

Since web apps are user-facing and often rely on sessions to keep track of each user, the appropriate partition key for caching is often stored within the session data, and needs to be retrieved before the cache lookup can take place. To help with this, MSAL Node provides the [DistributedCachePlugin](/javascript/api/@azure/msal-node/distributedcacheplugin) class, which implements the [ICachePlugin](https://azuread.github.io/microsoft-authentication-library-for-js/ref/interfaces/_azure_msal_node.ICachePlugin.html). An instance of `DistributedCachePlugin` requires:

* a **client interface** ([ICacheClient](/javascript/api/@azure/msal-node/icacheclient)), which implements `get` and `set` operations on the persistence server (Redis, MySQL etc.).
* a **partition manager** ([IPartitionManager](/javascript/api/@azure/msal-node/ipartitionmanager)), for reading from and writing to cache with respect to a given **session ID**.

Please refer to the [Web app using DistributedCachePlugin](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples/auth-code-distributed-cache) for a sample implementation.

## See also

See the samples below for more about how to handle caching in MSAL Node apps:

* [(PCA) Console app using MSAL Node Extensions](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/extensions/samples/msal-node-extensions/index.js)
* [(PCA) Dektop app using MSAL Node Extensions](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/extensions/samples/electron-webpack)
* [(CCA) Web app using DistributedCachePlugin](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples/auth-code-distributed-cache)
