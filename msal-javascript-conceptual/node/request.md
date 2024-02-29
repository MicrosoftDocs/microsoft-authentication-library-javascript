---
title: "Acquiring tokens in MSAL Node"
description: Learn how to acquire tokens in MSAL Node using the different OAuth 2.0 flows.
author: EmLauber
manager: CelesteDG
ms.author: emilylauber

ms.date: 11/28/2023
ms.service: msal
ms.subservice: msal-node
ms.topic: conceptual
ms.reviewer: dmwendia,cwerner, owenrichards, kengaderdus
#Customer intent: 
---

# Acquire tokens in MSAL Node

Since MSAL Node supports various authorization code grants, there is support for different public APIs per grant and the corresponding request. This article will walk you through the different public APIs available for each flow and the corresponding request type. It is strongly recommended that you implement the authorization code flow for your application.

## Authorization Code Flow

### Public APIs

- [getAuthCodeUrl()](/javascript/api/@azure/msal-node/publicclientapplication#@azure-msal-node-publicclientapplication-getauthcodeurl): This API is the first leg of the `authorization code grant` for MSAL Node. The request is of the type [AuthorizationUrlRequest](/javascript/api/@azure/msal-node/authorizationurlrequest).
The application is sent a URL that can be used to generate an `authorization code`. This URL can be opened in a browser of choice, where the user can input their credentials, and will be redirected back to the `redirectUri` (registered during the [app registration](/entra/identity-platform/scenario-desktop-app-registration)) with an `authorization code`. The `authorization code` can now be redeemed for a `token` with the following step. Note that if authorization code flow is being done for a public client application, [PKCE](https://tools.ietf.org/html/rfc7636) is recommended.

- [acquireTokenByCode()](/javascript/api/@azure/msal-node/publicclientapplication#@azure-msal-node-publicclientapplication-acquiretokenbycode): This API is the second leg of the `authorization code grant` for MSAL Node. The request constructed here should be of the type [AuthorizationCodeRequest](/javascript/api/@azure/msal-node/authorizationcodereques). The application passed the `authorization code` received as a part of the above step and exchanges it for a `token`. Not that if authorization code flow is being done for a public client application, [PKCE](https://tools.ietf.org/html/rfc7636) is recommended.

``` javascript

    const authCodeUrlParameters = {
        scopes: ["sample_scope"],
        redirectUri: "your_redirect_uri",
    };

    // get url to sign user in and consent to scopes needed for application
    cca.getAuthCodeUrl(authCodeUrlParameters).then((response) => {
        console.log(response);
    }).catch((error) => console.log(JSON.stringify(error)));

    const tokenRequest = {
        code: "authorization_code",
        redirectUri: "your_redirect_uri",
        scopes: ["sample_scope"],
    };

    // acquire a token by exchanging the code
    cca.acquireTokenByCode(tokenRequest).then((response) => {
        console.log("\nResponse: \n:", response);
    }).catch((error) => {
        console.log(error);
    });
```

## Device Code Flow

### Public APIs

- [acquireTokenByDeviceCode()](/javascript/api/@azure/msal-node/publicclientapplication#@azure-msal-node-publicclientapplication-acquiretokenbydevicecode): This API lets the application acquire a token with Device Code grant. The request is of the type [DeviceCodeRequest](/javascript/api/@azure/msal-node/devicecoderequest). This API acquires a `token` from the authority using OAuth2.0 device code flow. This flow is designed for devices that do not have access to a browser or have input constraints. The authorization server issues a DeviceCode object with a verification code, an end-user code, and the end-user verification URI. The DeviceCode object is provided through a callback, and the end-user should be instructed to use another device to navigate to the verification URI to input credentials. Since the client cannot receive incoming requests, it polls the authorization server repeatedly until the end-user completes input of credentials.

``` javascript
const msalConfig = {
    auth: {
        clientId: "your_client_id_here",
        authority: "your_authority_here",
    }
};

const pca = new msal.PublicClientApplication(msalConfig);

const deviceCodeRequest = {
    deviceCodeCallback: (response) => (console.log(response.message)),
    scopes: ["user.read"],
};

pca.acquireTokenByDeviceCode(deviceCodeRequest).then((response) => {
    console.log(JSON.stringify(response));
}).catch((error) => {
    console.log(JSON.stringify(error));
});

```

## Refresh Token Flow

### Public APIs

- [acquireTokenByRefreshToken](/javascript/api/@azure/msal-node/apiid#@azure-msal-node-apiid-acquiretokenbyrefreshtoken): This API acquires a token by exchanging the refresh token provided for a new set of tokens. The request is of the type [RefreshTokenRequest](/javascript/api/@azure/msal-node/refreshtokenrequest). The `refresh token` is never returned to the user in a response, but can be accessed from the user cache. It is recommended that you use `acquireTokenSilent()` for non-interactive scenarios. When using [acquireTokenSilent()](/javascript/api/@azure/msal-node/clientapplication#@azure-msal-node-clientapplication-acquiretokensilent), MSAL will handle the caching and refreshing of tokens automatically.

``` javascript
const config = {
    auth: {
        clientId: "your_client_id_here",
        authority: "your_authority_here",
    }
};

const pca = new msal.PublicClientApplication(config);

const refreshTokenRequest = {
    refreshToken: "",
    scopes: ["user.read"],
};

pca.acquireTokenByRefreshToken(refreshTokenRequest).then((response) => {
    console.log(JSON.stringify(response));
}).catch((error) => {
    console.log(JSON.stringify(error));
});
```

## Silent Flow

### Public APIs

- [acquireTokenSilent](/javascript/api/@azure/msal-node/clientapplication#@azure-msal-node-clientapplication-acquiretokensilent): This API acquires a token silently, in case cache is provided by the user, or when cache is created by preceding this call with any other interactive flow (eg: authorization code flow). The request is of the type [SilentFlowRequest](/javascript/api/@azure/msal-node/silentflowrequest). The `token` is acquired silently when a user specifies the account the token is requested for.

``` javascript
/**
 * Cache Plugin configuration
 */
const cachePath = "path_to_your_cache_file/msal_cache.json"; // Replace this string with the path to your valid cache file.

const readFromStorage = () => {
    return fs.readFile(cachePath, "utf-8");
};

const writeToStorage = (getMergedState) => {
    return readFromStorage().then(oldFile =>{
        const mergedState = getMergedState(oldFile);
        return fs.writeFile(cachePath, mergedState);
    })
};

const cachePlugin = {
    readFromStorage,
    writeToStorage
};

/**
 * Public Client Application Configuration
 */
const publicClientConfig = {
    auth: {
        clientId: "your_client_id_here",
        authority: "your_authority_here",
        redirectUri: "your_redirectUri_here",
    },
    cache: {
        cachePlugin
    },
};

/** Request Configuration */

const scopes = ["your_scopes"];

const authCodeUrlParameters = {
    scopes: scopes,
    redirectUri: "your_redirectUri_here",
};

const pca = new msal.PublicClientApplication(publicClientConfig);
const msalCacheManager = pca.getCacheManager();
let accounts;

pca.getAuthCodeUrl(authCodeUrlParameters)
    .then((response) => {
        console.log(response);
    }).catch((error) => console.log(JSON.stringify(error)));

const tokenRequest = {
    code: req.query.code,
    redirectUri: "http://localhost:3000/redirect",
    scopes: scopes,
};

pca.acquireTokenByCode(tokenRequest).then((response) => {
    console.log("\nResponse: \n:", response);
    return msalCacheManager.writeToPersistence();
}).catch((error) => {
    console.log(error);
});

// get Accounts
accounts = msalCacheManager.getAllAccounts();

// Build silent request
const silentRequest = {
    account: accounts[0], // You would filter accounts to get the account you want to get tokens for
    scopes: scopes,
};

// Acquire Token Silently to be used in MS Graph call
pca.acquireTokenSilent(silentRequest).then((response) => {
    console.log("\nSuccessful silent token acquisition:\nResponse: \n:", response);
    return msalCacheManager.writeToPersistence();
}).catch((error) => {
        console.log(error);
});
```

## Client Credentials Flow

### Public APIs

- [acquireTokenByClientCredential](/javascript/api/@azure/msal-node/confidentialclientapplication#@azure-msal-node-confidentialclientapplication-acquiretokenbyclientcredential): This API acquires a token using the confidential client application's credentials to authenticate (instead of impersonating a user) when calling another web service. In this scenario, the client is typically a middle-tier web service, a daemon service, or a back-end web application. For a higher level of assurance, the Microsoft identity platform also allows the calling service to use a certificate (instead of a shared secret) as a credential. The request is of the type [ClientCredentialRequest](/javascript/api/@azure/msal-node/clientcredentialrequest).

### Using secrets securely

Secrets should never be hardcoded. The dotenv npm package can be used to store secrets in a .env file (located in project's root directory) that should be included in .gitignore to prevent accidental uploads of the secrets.

``` javascript
import "dotenv/config"; // process.env now has the values defined in a .env file

const config = {
    auth: {
        clientId: "your_client_id_here",
        authority: "your_authority_here",
        clientSecret: process.env.clientSecret
    }
};

// Create msal application object
const cca = new msal.ConfidentialClientApplication(config);

// With client credentials flows permissions need to be granted in the portal by a tenant administrator.
// The scope is always in the format "<resource>/.default"
const clientCredentialRequest = {
    scopes: ["https://graph.microsoft.com/.default"], // replace with your resource
};

cca.acquireTokenByClientCredential(clientCredentialRequest).then((response) => {
    console.log("Response: ", response);
}).catch((error) => {
    console.log(JSON.stringify(error));
});
```

## On Behalf of Flow

- [acquireTokenOnBehalfOf](/javascript/api/@azure/msal-node/confidentialclientapplication#@azure-msal-node-confidentialclientapplication-acquiretokenonbehalfof): This API implements the On Behalf Of Flow, which is used when an application invokes a service/web API, which in turn needs to call another service/web API that uses any other authentication flow (device code, username/password, etc). The access token is acquired by the web API initially (by any of the web API flows), and the web API can then exchange this token for another token via OBO. The request is of the type [OnBehalfOfRequest](/javascript/api/@azure/msal-node/onbehalfofrequest)

Please look at the On Behalf Of flow [sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples/standalone-samples/on-behalf-of) for usage instructions:

* [WebAPI](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples/on-behalf-of/web-api/index.js) sample code
* [WebApp](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples/on-behalf-of/web-app/index.js) sample code

