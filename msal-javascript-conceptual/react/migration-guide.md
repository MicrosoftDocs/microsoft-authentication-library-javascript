---
title: Migration Guide from MSAL v1 to MSAL React and MSAL Browser
description: Learn how to migrate from msal.js v1 to MSAL React and MSAL Browser. This guide covers updating your app registration, installation, initialization, protecting components, acquiring an access token, acquiring an id token, updating redux store integration and reacting to events.
author: EmLauber
manager: CelesteDG

ms.service: msal
ms.subservice: msal-react
ms.topic: how-to
ms.date: 11/29/2023
ms.author: emilylauber
ms.reviewer: dmwendia, cwerner, owenrichards, kengaderdus
---

# Migration Guide from MSAL v1 to `@azure/msal-react` and `@azure/msal-browser`

This article provides an overview on migrating from MSAL v1 to `@azure/msal-react` and `@azure/msal-browser`. We recommend migration for improved performance and better security with the authorization code flow with PKCE and Conditional Access. Additionally, there's better single page application support. 

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/)
- An existing application registered in your Microsoft Entra tenant.

## Updating your app registration

The `@azure/msal-react` library, is a wrapper around `@azure/msal-browser` which implements the [Auth Code Flow with PKCE](/entra/identity-platform/v2-oauth2-auth-code-flow). This is a significant update from the MSAL v1 library which implements the [Implicit Flow](/entra/identity-platform/v2-oauth2-implicit-grant-flow).

You'll need to create a new app registration or update an existing one to use the new `redirectUri` type "SPA". Refer to [Single-page application: App registration](/entra/identity-platform/scenario-spa-app-registration#redirect-uri-msaljs-20-with-auth-code-flow) for more information.

## Installing `@azure/msal-react` and `@azure/msal-browser`

Both `@azure/msal-react` and its peer dependency `@azure/msal-browser` can be installed from npm. Its important to uninstall your old MSAL package. Open a terminal and run the following commands.

```console
npm uninstall msal
npm install @azure/msal-react @azure/msal-browser
```

## Upgrading from `react-aad-msal`

If your app currently uses [React Microsoft Entra MSAL](https://www.npmjs.com/package/react-aad-msal) for authentication and you are looking to migrate to `@azure/msal-react` this section will outline the differences between the two libraries and some of the changes you need to make. React Microsoft Entra MSAL is a 3rd party library and MSAL React was built from the ground up there may be some edge cases that are not covered or not supported by MSAL React.

The following are features supported in `react-aad-msal` which are not supported in `@azure/msal-react`:

- Verifying IdToken expiration before rendering protected components & automatic refresh of expired IdTokens
- Out of the box support for redux store (alternative below)

For other cases possible with `react-aad-msal` but no lonter possible with `@azure/msal-react`, open an issue in the [microsoft-authentication-library-for-js](https://github.com/AzureAD/microsoft-authentication-library-for-js/issues/new/choose) GitHub repo.

### Initialization

In `react-aad-msal` you initialize your MSAL instance by creating an `MsalAuthProvider` object which is later passed down to the `AzureAD` component.

```javascript
import { MsalAuthProvider } from "react-aad-msal";

const authProvider = new MsalAuthProvider(config, authenticationParameters, options);
```

In `@azure/msal-react` you initialize your MSAL instance using `PublicClientApplication` exported from `@azure/msal-browser` which is then passed down to the `MsalProvider` component exported from `@azure/msal-react`. The config options are largely similar between `msal` and `@azure/msal-browser`, however you can refer to the [Configuration type](/javascript/api/@azure/msal-browser/configuration) for the most up to date config options.

The `authenticationParameters` and `options` parameters used in `react-aad-msal` are not used in `@azure/msal-react`, though similar functionality can be achieved on individual components. This will be explained later on in this document.

`@azure/msal-react` uses the [React Context API](https://reactjs.org/docs/context.html) to make `PublicClientApplication` and authentication state available throughout your entire component tree.

```javascript
import { PublicClientApplication } from "@azure/msal-browser";
import { MsalProvider } from "@azure/msal-react";

const pca = new PublicClientApplication(config);

function App() {
    return (
        <MsalProvider instance={pca}>
            <YourAppComponents />
        </MsalProvider>
    );
}
```

General notes about the `MsalProvider` component:

- All components that need access to authentication state or hooks/components exposed by `@azure/msal-react` must have an `MsalProvider` higher up in the component tree, therefore it is recommended that `MsalProvider` be rendered as close to the root as possible.
- Your app should not render more than 1 `MsalProvider` component on any given page.
- We don't recommend initializing `PublicClientApplication` inside a component due to the possiblity of re-renders

### Protecting your components

In `react-aad-msal` components are protected using the `AzureAD` component or `withAuthentication` HOC which wraps your component with `AzureAD` under the hood. The `AzureAD` component will only render child components if a user is authenticated and optionally initiate a login if no user is authenticated. The options used for login (e.g. Scopes, whether to use popup or redirect, etc.) are specified earlier, when creating the `authProvider` prop.

```javascript
import { MsalAuthProvider } from "react-aad-msal";

const authProvider = new MsalAuthProvider(config, authenticationParameters, options);

function App() {
    return (
        <AzureAD provider={authProvider} forceLogin={true}>
            <span>Only authenticated users can see me.</span>
        </AzureAD>
    );
}
```

`@azure/msal-react`, on the other hand, gives developers more control over what they want to display to whom.

- The `AuthenticatedTemplate` component will render children if a user is authenticated
- The `UnauthenticatedTemplate` component will render children if a user is not authenticated
- The `MsalAuthenticationTemplate` component will automatically initiate a login if a user is unauthenticated and then render children once a user is authenticated.

```javascript
import { PublicClientApplication, InteractionType } from "@azure/msal-browser";
import { MsalProvider, AuthenticatedTemplate, UnauthenticatedTemplate, MsalAuthenticationTemplate } from "@azure/msal-react";

const pca = new PublicClientApplication(config);

function App() {
    return (
        <MsalProvider instance={pca}>
            <AuthenticatedTemplate>
                <span>Only authenticated users can see me.</span>
            </AuthenticatedTemplate>
            <UnauthenticatedTemplate>
                <span>Only unauthenticated users can see me.</span>
            </UnauthenticatedTemplate>
            <MsalAuthenticationTemplate interactionType={InteractionType.Popup} authenticationRequest={request}>
                <span>Only authenticated users can see me. Unauthenticated users will get a popup asking them to login first.</span>
            </MsalAuthenticationTemplate>
        </MsalProvider>
    );
}
```

Additionally, if you prefer to take a hooks based approach `@azure/msal-react` provides several hooks you can use to achieve similar results. These are just some basic examples, and you can refer to [MSAL React hooks](./hooks.md) for more information.

```javascript
import { PublicClientApplication, InteractionType } from "@azure/msal-browser";
import { MsalProvider, useIsAuthenticated, useMsalAuthentication } from "@azure/msal-react";

const pca = new PublicClientApplication(config);

function App() {
    return (
        <MsalProvider instance={pca}>
            <ExampleComponent />
        </MsalProvider>
    );
}

function ExampleComponent() {
    const isAuthenticated = useIsAuthenticated();
    const { error } = useMsalAuthentication(InteractionType.Popup, request); // Will initiate a popup login if user is unauthenticated

    if (isAuthenticated) {
        return <span>Only authenticated users can see me.</span>
    } else if (error) {
        return <span>An error occurred during login!</span>
    } else {
        return <span>Only unauthenticated users can see me.</span>
    }
}
```

### Acquiring an access token

`react-aad-msal` exposes a `getAccessToken` method you can use to obtain an access token before calling an API.

```javascript
import { MsalAuthProvider } from "react-aad-msal";

const authProvider = new MsalAuthProvider(config, authenticationParameters, options);
const accessToken = authProvider.getAccessToken();
```

When using `@azure/msal-react` and `@azure/msal-browser` you will call `acquireTokenSilent` on the `PublicClientApplication` instance.

If you need to obtain an access token inside a component or hook that lives under `MsalProvider` you can use the `useMsal` hook to get the objects you need.

```javascript
import { useState } from "react";
import { useMsal } from "@azure/msal-react";
import { InteractionRequiredAuthError } from "@azure/msal-browser";

function useAccessToken() {
    const { instance, accounts } = useMsal();
    const [accessToken, setAccessToken] = useState(null);

    if (accounts.length > 0) {
        const request = {
            scopes: ["User.Read"],
            account: accounts[0]
        };
        instance.acquireTokenSilent(request).then(response => {
            setAccessToken(response.accessToken);
        }).catch(error => {
            // acquireTokenSilent can fail for a number of reasons, fallback to interaction
            if (error instanceof InteractionRequiredAuthError) {
                instance.acquireTokenPopup(request).then(response => {
                    setAccessToken(response.accessToken);
                });
            }
        });
    }

    return accessToken;
}
```

If you need to obtain an access token outside the context of `MsalProvider` you can use the `PublicClientApplication` instance directly and call `getAllAccounts()` to get the account object.

> [!IMPORTANT]
> Only attempt silent token acquisition outside the context of `MsalProvider`. You should not call an interactive method (redirect or popup) outside the context of `MsalProvider`.

The example below shows initialization of `PublicClientApplication` for demonstration purposes. `PublicClientApplication` should only be initialized once per page load and you should use the same instance here that you provide to `MsalProvider`.

```javascript
import { PublicClientApplication } from "@azure/msal-browser";

const pca = new PublicClientApplication(config);
const accounts = pca.getAllAccounts();

async function getAccessToken() {
    if (accounts.length > 0) {
        const request = {
            scopes: ["User.Read"],
            account: accounts[0]
        }
        const accessToken = await pca.acquireTokenSilent(request).then((response) => {
            return response.accessToken;
        }).catch(error => {
            // Do not fallback to interaction when running outside the context of MsalProvider. Interaction should always be done inside context.
            console.log(error);
            return null;
        });

        return accessToken;
    }

    return null;
}
```

### Acquiring an ID token

`react-aad-msal` exposed a `getIdToken` function to get or renew an idToken.

```javascript
import { MsalAuthProvider } from "react-aad-msal";

const authProvider = new MsalAuthProvider(config, authenticationParameters, options);
const token = await authProvider.getIdToken();
const idToken = token.idToken.rawIdToken;
```

You may also be familiar with the pattern of requesting your `clientId` as the only scope in order to retrive an idToken.
This is no longer a supported pattern in `@azure/msal-browser`.

In `@azure/msal-react` and `@azure/msal-browser` all token calls will return both an access token and an id token and all access token renewals will also renew the id token.

If you need to obtain an id token inside a component or hook that lives under `MsalProvider` you can use the `useMsal` hook to get the objects you need.

```javascript
import { useState } from "react";
import { useMsal } from "@azure/msal-react";

function useIdToken() {
    const { instance, accounts } = useMsal();
    const [idToken, setIdToken] = useState(null);

    if (accounts.length > 0) {
        const request = {
            scopes: ["openid"],
            account: accounts[0]
        };
        instance.acquireTokenSilent(request).then(response => {
            setIdToken(response.idToken);
        }).catch(error => {
            // acquireTokenSilent can fail for a number of reasons, fallback to interaction
            if (error instanceof InteractionRequiredAuthError) {
                instance.acquireTokenPopup(request).then(response => {
                    setIdToken(response.idToken);
                });
            }
        });
    }

    return idToken;
}
```

If you need to obtain an id token outside the context of `MsalProvider` you can use the `PublicClientApplication` instance directly and call `getAllAccounts()` to get the account object.

> [!IMPORTANT]
> Only attempt silent token acquisition outside the context of `MsalProvider`. You should not call an interactive method (redirect or popup) outside the context of `MsalProvider`.

The example below shows initialization of `PublicClientApplication` for demonstration purposes. `PublicClientApplication` should only be initialized once per page load and you should use the same instance here that you provide to `MsalProvider`.

```javascript
import { PublicClientApplication } from "@azure/msal-browser";

const pca = new PublicClientApplication(config);
const accounts = pca.getAllAccounts();

async function getIdToken() {
    if (accounts.length > 0) {
        const request = {
            scopes: ["openid"],
            account: accounts[0]
        }
        const idToken = await pca.acquireTokenSilent(request).then((response) => {
            return response.idToken;
        }).catch (error => {
            // Do not fallback to interaction when running outside the context of MsalProvider. Interaction should always be done inside context.
            console.log(error);
            return null;
        });

        return idToken
    }

    return null;
}
```

### Updating redux store integration / reacting to events

`react-aad-msal` provided out of the box integration with a redux store by dispatching actions when events such as login or logout occurred.
`@azure/msal-react` does not provide this feature, however, similar functionality can be achieved using the [event api](../browser/events.md) exposed by `@azure/msal-browser`.

You can register an event callback that will be called each time an event is broadcast (e.g. `LOGIN_SUCCESS`). Your callback function can inspect the event and do something with the payload. If you would like to continue using your existing redux store you can register an event callback that dispatches actions to your store.

```javascript
import { PublicClientApplication, EventType } from "@azure/msal-browser";
import { store } from "your-redux-store-implementation";

const msalInstance = new PublicClientApplication(config);

const callbackId = msalInstance.addEventCallback((message: EventMessage) => {
    if (message.eventType === EventType.LOGIN_SUCCESS) {
        store.dispatchAction({type: "AAD_LOGIN_SUCCESS", payload: message.payload});
    }
});
```

The payloads may differ between `msal` v1 and `@azure/msal-browser` so you may need to make some adjustments if your application relies on specific fields or the object shape. Our typedocs contain the most up to date list of [event types](/javascript/api/@azure/msal-browser/eventtype) and [payload types](/javascript/api/@azure/msal-browser/eventpayload) and you can find the mapping between the two in the [event doc](../browser/events.md).

## See also

- [MSAL React Type Aliases](/javascript/api/@azure/msal-react/#typeAliases)
