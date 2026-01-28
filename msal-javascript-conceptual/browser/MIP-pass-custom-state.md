---
title: Pass custom state in authentication requests (MSAL.js)
description: Learn how to pass a custom state parameter value in authentication request using the Microsoft Authentication Library for JavaScript (MSAL.js).
author: Dickson-Mwendia
manager: Dougeby
ms.author: dmwendia
ms.date: 01/28/2026
ms.reviewer: Jo.Arroyo
ms.service: msal
ms.subservice: msal-js
ms.topic: how-to
#Customer intent: As an application developer, I want to learn about passing custom state in authentication requests so I can create more robust applications.
---

# Pass custom state in authentication requests using MSAL.js

The *state* parameter, as defined by OAuth 2.0, is included in an authentication request and is also returned in the token response to prevent cross-site request forgery attacks. By default, the Microsoft Authentication Library for JavaScript (MSAL.js) passes a randomly generated unique *state* parameter value in the authentication requests.

The state parameter can also be used to encode information of the app's state before redirect. You can pass a reference to the user's state in the app, such as an identifier for the page or view they were on, as input to this parameter. The MSAL.js library allows you to pass your custom state as state parameter in the [Request](https://azuread.github.io/microsoft-authentication-library-for-js/ref/modules/_azure_msal_browser.html#redirectrequest) object.

> [!IMPORTANT]
> For security and privacy, do not put URLs or other sensitive data directly in the state parameter. Instead, use a key or identifier that corresponds to data stored in browser storage, such as localStorage or sessionStorage. This approach lets your app securely reference the necessary data after authentication. For example, you can store the actual URL in sessionStorage and pass only the storage key in the state parameter.

For example:

```javascript
import {PublicClientApplication} from "@azure/msal-browser";

const myMsalObj = new PublicClientApplication({
    clientId: "ENTER_CLIENT_ID_HERE"
});

let loginRequest = {
    scopes: ["user.read"],
    state: "state_key"
}

myMSALObj.loginRedirect(loginRequest);
```

The passed in state is appended to the unique GUID set by MSAL.js when sending the request. When the response is returned, MSAL.js checks for a state match and then returns the custom passed in state in the [Response](https://azuread.github.io/microsoft-authentication-library-for-js/ref/modules/_azure_msal_common.html#authenticationresult) object as `state`.

To learn more, read about [building a single-page application (SPA)](/entra/identity-platform/scenario-spa-overview) using MSAL.js.
