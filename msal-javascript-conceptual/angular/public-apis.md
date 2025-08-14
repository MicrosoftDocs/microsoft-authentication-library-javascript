---
title: Commonly used public APIs in MSAL Angular
description: Commonly used public APIs in MSAL Angular
author: Dickson-Mwendia
manager: Dougeby
ms.service: msal
ms.subservice: msal-angular
ms.topic: article
ms.date: 05/21/2025
ms.author: dmwendia
ms.reviewer: cwerner, owenrichards, kengaderdus
---

# Commonly used public APIs in MSAL Angular

Before you start here, make sure you understand how to [initialize the application object](initialization.md).

The login APIs in MSAL retrieve an `authorization code` which can be exchanged for an [ID token](/entra/identity-platform/id-tokens) for a signed in user, while consenting scopes for an additional resource, and an [access token](/entra/identity-platform/access-tokens) containing the user consented scopes to allow your app to securely call the API. Learn more about [ID tokens](/entra/identity-platform/id-tokens).

## Public APIs

`@azure/msal-angular` exposes the following, along with their configurations. See the [library references](https://azuread.github.io/microsoft-authentication-library-for-js/ref/modules/_azure_msal_angular.html) for properties and methods.  

1. [`MsalService`](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/src/msal.service.ts/)
1. [`MsalGuard`](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/src/msal.guard.ts/)
    * [`MsalGuardConfiguration`](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/src/msal.guard.config.ts/)
1. [`MsalInterceptor`](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/src/msal.interceptor.ts/)
    * [`MsalInterceptorConfiguration`](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/src/msal.interceptor.config.ts/)
1. [`MsalBroadcastService`](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/src/msal.broadcast.service.ts/)
1. [`MsalModule`](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/src/msal.module.ts/)


The login and acquire token functions using Angular observables are found on the [IMsalService](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/src/IMsalService.ts/).

`@azure/msal-angular` also exposes the following:

1. [`MsalRedirectComponent`](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/src/msal.redirect.component.ts): Used for handling redirects. See the [redirect doc](redirects.md) for more details.
1. [`MsalCustomNavigationClient`](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/src/msal.navigation.client.ts): Used for client-side navigation. See the [performance doc](performance.md) for more details.

Additional functions from `@azure/msal-browser` are found on [`IPublicClientApplication`](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/src/app/IPublicClientApplication.ts), with corresponding documentation [here](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/login-user.md).
