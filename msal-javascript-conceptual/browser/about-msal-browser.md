---
title: About MSAL Browser
description: Learn how to use MSAL Browser in your JavaScript applications
author: EmLauber
manager: CelesteDG

ms.service: msal
ms.subservice: msal-js
ms.topic: conceptual
ms.date: 01/10/2024
ms.author: emilylauber
ms.reviewer: dmwendia, cwerner, owenrichards, kengaderdus
---


# Using MSAL Browser in your JavaScript applications

The MSAL library for JavaScript enables client-side JavaScript applications to authenticate users using [Microsoft Entra ID](/entra/identity-platform/vv2-overview.md) work and school accounts, Microsoft personal accounts (MSA) and social identity providers like Facebook, Google, LinkedIn, Microsoft accounts, etc. through [Azure AD B2C](/azure/active-directory-b2c/active-directory-b2c-overview.md#identity-providers) service. It also enables your app to get tokens to access [Microsoft Cloud](https://www.microsoft.com/enterprise) services such as [Microsoft Graph](https://graph.microsoft.io).

The `@azure/msal-browser` package uses the [`@azure/msal-common` package](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-common) as a dependency to enable authentication in JavaScript Single-Page Applications without backend servers. This version of the library uses the OAuth 2.0 Authorization Code Flow with PKCE and does NOT support the implicit flow.

## Prerequisites

-   `@azure/msal-browser` is meant to be used in [Single-Page Application scenarios](/entra/identity-platform/vscenario-spa-overview.md).

-   Before using `@azure/msal-browser` you will need to [register a Single Page Application in Microsoft Entra ID](/entra/identity-platform/scenario-spa-app-registration.md) to get a valid `clientId` for configuration, and to register the routes that your app will accept redirect traffic on.

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

### Building the package

To build the `@azure/msal-browser` library, you can do the following:

```bash
// Change to the msal-browser package directory
cd lib/msal-browser/
// To run build only for browser package
npm run build
```

To build both the `@azure/msal-browser` library and `@azure/msal-common` libraries, you can do the following:

```bash
// Change to the msal-browser package directory
cd lib/msal-browser/
// To run build only for browser package
npm run build:all
```

### Running Tests

`@azure/msal-browser` uses [jest](https://jestjs.io) to run unit tests.

```bash
// To run tests
npm test
// To run tests with code coverage
npm run test:coverage
```

## Framework Wrappers

If you are using a framework such as Angular or React you may be interested in using one of our wrapper libraries:

-   Angular: [`@azure/msal-angular` v2](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-angular)
-   React: [`@azure/msal-react`](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-react)
