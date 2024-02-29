---
title: Accounts in MSAL Node
description: Learn how to use the different APIs in MSAL Node to access cached accounts.
author: EmLauber
manager: CelesteDG
ms.author: emilylauber

ms.service: msal
ms.subservice: msal-node
ms.date: 10/26/2023
ms.topic: conceptual
ms.reviewer: dmwendia,cwerner, owenrichards, kengaderdus
#Customer intent: As a developer, I want to learn how to use the different APIs in MSAL Node to access cached accounts.
---

# Accounts in MSAL Node

This article shows you how to use the `msal-node` library to access cached accounts in your Node.js application. The APIs, `getAllAccounts()`, `getAccountByHomeId()`, and `getAccountByLocalId()` are covered 

## Prerequisites

- [Node.js](https://nodejs.org/en/download/)

## Usage

The `msal-node` library provides the following different APIs to access cached accounts:

* `getAllAccounts()`: returns all the accounts currently in the cache. An application must choose an account to acquire tokens silently.
* `getAccountByHomeId()`: receives a `homeAccountId` string and returns the matching account from the cache.
* `getAccountByLocalId()`: receives a `localAccountId` string and returns the matching account from the cache.

The following are usage examples for each API:

### `getAllAccounts`

For a multiple accounts scenario:

```javascript
// Initiates Acquire Token Silent flow
function callAcquireTokenSilent()
    // Find all accounts
    const msalTokenCache = myMSALObj.getTokenCache();
    const cachedAccounts = await msalTokenCache.getAllAccounts();

    // Account selection logic would go here

    const account = .... // Select Account code

    // Build silent request after account is selected
    const silentRequest = {
        account: account,
        scopes: scopes,
    };

    // Acquire Token Silently to be used in MS Graph call
    myMSALObj.acquireTokenSilent(silentRequest)
        .then((response) => {
            // Successful response handling
        })
        .catch((error) => {
            // Error handling
        });
});
```

### `getAccountByHomeId` and `getAccountByLocalId`

For a single account scenario, the `homeAccountId` or `localAccountId` must be obtained from the initial `AuthResponse` object received from a non-silent authorization flow, such as the `Auth Code` flow.

```javascript

// Initialize global homeAccountId variable, ideally stored in application state
let homeAccountId = null; // Same for localAccountId

// Get MSAL Token Cache from MSAL Client Applicaiton object
const msalTokenCache = myMSALObj.getTokenCache();

// Initial token acquisition, second leg of Auth Code flow
function getTokenAuthCode() {
    const tokenRequest = {
        code: req.query.code,
        redirectUri: "http://localhost:3000/redirect",
        scopes: scopes,
    };

    myMSALObj.acquireTokenByCode(tokenRequest).then((response) => {
        // Home account ID or local account ID to be used to find the right account before acquireTokenSilent
        homeAccountId = response.account.homeAccountId; // Same for localAccountId
        .
        .
        .
        // Handle successful token response
    }).catch((error) => {
        // Handle token request error
    });
}
```

Once the account and tokens are cached and the application state holds the `homeAccountId` or `localAccountId` string, `getAccountByHomeId` and `getAccountByLocalId` can be used before an `acquireTokenSilent` call:

```javascript
async function getResource() {
    // Find account using homeAccountId or localAccountId built after receiving auth code token response
    const account = await msalTokenCache.getAccountByHomeId(app.locals.homeAccountId); // alternativley: await msalTokenCache.getAccountByLocalId(localAccountId) if using localAccountId

    // Build silent request
    const silentRequest = {
        account: account,
        scopes: scopes,
    };
    // Acquire Token Silently to be used in Resource API call
    pca.acquireTokenSilent(silentRequest)
        .then((response) => {
            // Handle successful resource API response
        })
        .catch((error) => {
            // Handle resource API request error
        });
}
```

For a multiple accounts scenario, please should modify the [sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/samples/msal-node-samples/silent-flow/index.js) (in `/graphCall` route) to list all cached accounts and choose a specific account. You may also need to customize the related view templates and `handlebars` template params.

## See also

* The current msal-node silent-flow [sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples/silent-flow) has a working single account scenario that uses `getAccountByHomeId()`.
