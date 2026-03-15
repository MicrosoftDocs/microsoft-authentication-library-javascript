---
title: About MSAL Browser
description: Learn how to use MSAL Browser in your JavaScript applications
author: Dickson-Mwendia
manager: Dougeby
ms.service: msal
ms.subservice: msal-js
ms.topic: concept-article
ms.date: 03/15/2026
ms.author: dmwendia
ms.reviewer: kengaderdus
#Customer intent: As a JavaScript developer, I want to understand how to install and set up MSAL Browser so that I can add authentication to my single-page application.
---

# Using MSAL Browser in your JavaScript applications

The MSAL library for JavaScript enables client-side JavaScript applications to authenticate users using [Microsoft Entra ID](/entra/identity-platform/v2-overview) work and school accounts, Microsoft personal accounts (MSA) and social identity providers like Facebook, Google, LinkedIn, Microsoft accounts, etc. through [Azure AD B2C](/azure/active-directory-b2c/overview#identity-providers) service. It also enables your app to get tokens to access [Microsoft Cloud](https://www.microsoft.com/enterprise) services such as [Microsoft Graph](https://graph.microsoft.com).

The `@azure/msal-browser` package enables authentication in JavaScript single-page applications using the OAuth 2.0 Authorization Code Flow with PKCE. It doesn't support the implicit flow. The current version is MSAL.js v5.x. If you're using an older version, see the [migration guides](./v4-migration.md) to upgrade.

## Prerequisites

-   `@azure/msal-browser` is meant to be used in [Single-Page Application scenarios](/entra/identity-platform/scenario-spa-overview).

-   Before using `@azure/msal-browser` you will need to [register a Single Page Application in Microsoft Entra ID](/entra/identity-platform/scenario-spa-app-registration) to get a valid `clientId` for configuration, and to register the routes that your app will accept redirect traffic on.

## Key features

MSAL Browser provides the following capabilities for your single-page applications:

- [Sign in users with popup or redirect flows](./login-user.md)
- [Acquire tokens silently from cache or via refresh](./acquire-token.md)
- [Support for COOP (Cross-Origin-Opener-Policy) popup flows](./login-user.md)
- [MCP (Model Context Protocol) authentication](./mcp.md)
- [Nested App Authentication (NAA) for Microsoft 365 apps](./nested-app-auth.md)
- [Device-bound tokens via platform broker (WAM)](./device-bound-tokens.md)
- [AES-GCM encrypted token cache in localStorage](./caching.md)
- [Proof of Possession (PoP) tokens](./access-token-proof-of-possession.md)
- [Single sign-on across tabs and applications](./single-sign-on.md)

## Installation

### Via NPM

```javascript
npm install @azure/msal-browser
```

## Samples

The [`msal-browser-samples` folder](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-browser-samples) contains sample applications for our libraries.

More instructions to run the samples can be found in the [`README.md` file](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/samples/msal-browser-samples/VanillaJSTestApp2.0/Readme.md) of the VanillaJSTestApp2.0 folder.

More advanced samples backed with a tutorial can be found in the [Azure Samples](https://github.com/Azure-Samples) space on GitHub:

-   [JavaScript SPA calling Express.js web API](https://github.com/Azure-Samples/ms-identity-javascript-tutorial/tree/main/3-Authorization-II/1-call-api)
-   [JavaScript SPA calling Microsoft Graph via Express.js web API using on-behalf-of flow](https://github.com/Azure-Samples/ms-identity-javascript-tutorial/tree/main/4-AdvancedGrants/1-call-api-graph)
-   [Deployment tutorial for Azure App Service and Azure Storage](https://github.com/Azure-Samples/ms-identity-javascript-tutorial/tree/main/5-Deployment)

## Framework Wrappers

If you're using a framework such as Angular or React you may be interested in using one of our wrapper libraries:

-   Angular: [`@azure/msal-angular` (current: v5.1.1)](../angular/initialization.md)
-   React: [`@azure/msal-react` (current: v5.0.6)](../react/getting-started.md)

## Next steps

- [Initialize your application](./initialization.md)
- [Configure MSAL Browser](./configuration.md)
- [Sign in users](./login-user.md)
