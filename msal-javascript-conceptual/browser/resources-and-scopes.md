---
title:  Resources and Scopes
description: Learn about accessing resources and the scopes included in token requests
author: EmLauber
manager: CelesteDG

ms.service: msal
ms.subservice: msal-js
ms.topic: conceptual
ms.date: 01/10/2024
ms.author: emilylauber
ms.reviewer: dmwendia, cwerner, owenrichards, kengaderdus
---

# Resources and scopes

Microsoft identity platform employs a *scope-centric* model to access resources. Here, a *resource* refers to any application that can be a recipient of an **Access Token** (such as [MS Graph API](/graph/overview) or your own web API), and a *scope* (*aka* "permission") refers to any aspect of a resource that an **Access Token** grants rights.

**Access Token** requests in **MSAL.js** are meant to be *per-resource-per-scope(s)*. This means that an **Access Token** requested for resource **A** with scope `scp1`:

- cannot be used for accessing resource **A** with scope `scp2`, and,
- cannot be used for accessing resource **B** of any scope.

The intended recipient of an **Access Token** is represented by the `aud` claim; in case the value for the `aud` claim does not match the resource [APP ID URI](/entra/identity-platform/scenario-protected-web-api-app-registration.md), the token should be considered invalid. Likewise, the permissions that an **Access Token** grants is represented by the `scp` claim. See [Access Token claims](/entra/identity-platform/access-tokens#payload-claims.md) for more information.

## Default scopes

By default MSAL.js will add the `openid`, `profile` and `offline_access` scopes to every request. These scopes are required in order to receive a refresh token and the id token claims used to populate the account object with the user's information.

## Working with multiple resources

When you have to access multiple resources, initiate a separate token request for each:

 ```javascript
 // "User.Read" stands as shorthand for "graph.microsoft.com/User.Read"
 const graphToken = await msalInstance.acquireTokenSilent({
      scopes: [ "User.Read" ]
 });
 const customApiToken = await msalInstance.acquireTokenSilent({
      scopes: [ "api://<myCustomApiClientId>/My.Scope" ]
 });
 ```

Bear in mind that you *can* request multiple scopes for the same resource (e.g. `User.Read`, `User.Write` and `Calendar.Read` for **MS Graph API**).

 ```javascript
 const graphToken = await msalInstance.acquireTokenSilent({
      scopes: [ "User.Read", "User.Write", "Calendar.Read" ] // all MS Graph API scopes
 });
 ```

In case you *erroneously* pass multiple resources in your token request, the token you will receive will only be issued for the first resource.

 ```javascript
 // you will only receive a token for MS GRAPH API's "User.Read" scope here
 const myToken = await msalInstance.acquireTokenSilent({
      scopes: [ "User.Read", "api://<myCustomApiClientId>/My.Scope" ]
 });
 ```

## Dynamic scopes and incremental consent

In **Microsoft Entra ID**, the scopes (*permissions*) set directly on the application registration are called **static scopes**. Other scopes that are only defined within the code are called **dynamic scopes**. This has implications on the **login** (i.e. *loginPopup*, *loginRedirect*) and **acquireToken** (i.e. *acquireTokenPopup*, *acquireTokenRedirect*, *acquireTokenSilent*) methods of **MSAL.js**. Consider:

 ```javascript
  const loginRequest = {
       scopes: [ "openid", "profile", "User.Read" ]
  };
  const tokenRequest = {
       scopes: [ "Mail.Read" ]
  };
  // will return an ID Token and an Access Token with scopes: "openid", "profile" and "User.Read"
  msalInstance.loginPopup(loginRequest);
  // will fail and fallback to an interactive method prompting a consent screen
  // after consent, the received token will be issued for "openid", "profile" ,"User.Read" and "Mail.Read" combined
  msalInstance.acquireTokenSilent(tokenRequest);
 ```

In the code snippet above, the user will be prompted for consent once they authenticate and receive an **ID Token** and an **Access Token** with the scope `User.Read`. Later, if they request an **Access Token** for `User.Read`, they will not be asked for consent again (in other words, they can acquire a token *silently*).

On the other hand, the user did not consent to `Mail.Read` at the authentication stage, therefore, will be asked for consent when requesting an **Access Token** for `Mail.Read` scope. The token received will contain all the previously consented scopes (for that specific resource), hence the term *incremental consent*.

Consider a slightly different case:

 ```javascript
  const loginRequest = {
       scopes: [ "openid", "profile", "User.Read" ],
       extraScopesToConsent: [ "api://<myCustomApiClientId>/My.Scope"]
  };
  const tokenRequest = {
       scopes: [ "Mail.Read" ]
  };
  const anotherTokenRequest = {
       scopes: [ "api://<myCustomApiClientId>/My.Scope" ]
  }
  // will return an ID Token and an Access Token with scopes: "openid", "profile" and "User.Read"
  msalInstance.loginPopup(loginRequest);
  // will fail with InteractionRequiredError due to lack of consent for "Mail.Read" scope. You should fallback to an interactive method in this case.
  msalInstance.acquireTokenSilent(tokenRequest);
  // will succeed and return an Access Token with scope "api://<myCustomApiClientId>/My.Scope"
  msalInstance.acquireTokenSilent(anotherTokenRequest);
 ```

In the code snippet above, even though the user consents to both `User.Read` and `api://<myCustomApiClientId>/My.Scope` scopes, they will only receive an **Access Token** for **MS Graph API**, in accordance with *per-resource-per-scope(s)* principle. However, since they already consented to `api://<myCustomApiClientId>/My.Scope`, they can acquire an **Access Token** for that resource/scope *silently* later on.

## Consent lifetime

In Microsoft Entra ID, consent lives beyond the lifetime of the application. This means that, when you request an **Access Token** for a resource, all the scopes you have previously consented to for that resource will be returned, regardless of what scope was requested at the time. In other words, if you consent to `User.Read` and `Mail.Read` today and run a new instance of your application tomorrow requesting an **Access Token** for `User.Read` only, you will still receive a token issued for **both** `User.Read` and `Mail.Read`. For more information, refer to [Permissions and Consent](/entra/identity-platform/v2-permissions-and-consent#using-permissions.md).
