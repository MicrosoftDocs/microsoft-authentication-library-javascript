---
title: Handle errors and exceptions in MSAL Node
description: Learn how to handle use MSAL React with class components. covering initialization, protecting components, accessing MSAL React context and logging in.
author: EmLauber
manager: CelesteDG

ms.service: msal
ms.subservice: msal-react
ms.topic: conceptual
ms.date: 11/29/2023
ms.author: emilylauber
ms.reviewer: dmwendia,cwerner, owenrichards, kengaderdus
---

# Handle errors and exceptions in MSAL Node

This article covers how to handle some of the errors and exceptions that can be thrown by MSAL React. We'll cover BrowserConfigurationAuthErrors, and BrowserAuthErrors and provide some troubleshooting steps to help you resolve them.

## BrowserConfigurationAuthErrors

BrowserConfigurationAuthErrors are used to represent errors that occur due to incorrect configuration of the MSAL authentication parameters, and can occur when initializing a `PublicClientApplication` or `ConfidentialClientApplication` instance. If the configuration object passed to the constructor does not meet the library's requirements, you don't provide a valid client ID or authority, or if you provide an invalid redirect URI, you might encounter this type of error

### `stubbed_public_client_application_called`

This error means that a method has been called on a `PublicClientApplication` instance that has been stubbed out or incorrectly initialized, when you try to use an msal component or hook without an `MsalProvider` higher up in the component tree. All hooks and components make use of the [React Context API](https://reactjs.org/docs/context.html) and require a provider. You may get a message similar to `Stub instance of PublicClientApplication was called. If using @azure/msal-react, please ensure context is not used without a provider`.

The following snippet will throw this error because the `useMsal` hook is used outside the context of `MsalProvider`.

```javascript
import { useMsal, MsalProvider } from "@azure/msal-react";
import { PublicClientApplication } from "@azure/msal-browser";

const pca = new PublicClientApplication(config);

function App() {
    const { accounts } = useMsal();

    return (
        <MsalProvider instance={pca}>
            <YourAppComponent>
        </ MsalProvider>
    )
}
```

To resolve the error you should refactor the preceding snippet so that the `useMsal` hook is called in a component underneath `MsalProvider`. The correct implementation is shown in the following snippet.

```javascript
import { useMsal, MsalProvider } from "@azure/msal-react";
import { PublicClientApplication } from "@azure/msal-browser";

const pca = new PublicClientApplication(config);

function ExampleComponent () {
    const { accounts } = useMsal();

    return <YourAppComponent />;
};

function App() {
    return (
        <MsalProvider instance={pca}>
            <ExampleComponent />
        </ MsalProvider>
    )
}
```

## BrowserAuthErrors

BrowserAuthErrors are used to represent errors that occur during the authentication process in a browser environment. Such occurrences can be when a user tries to initiate a new login request while a previous request is still being processed. To resolve these errors, you should ensure that each interaction has completed before starting a new one.

### `Interaction_in_progress`

This error is thrown when an interactive API (`loginPopup`, `loginRedirect`, `acquireTokenPopup`, `acquireTokenRedirect`) is invoked while another interactive API is still in progress. The login and acquireToken APIs are async so you will need to ensure that the resulting promises have resolved before invoking another one. You may get an error message similar to `Interaction is currently in progress. Please ensure that this interaction has been completed before calling an interactive API.`

In `@azure/msal-react` there are 2 scenarios when this can happen:

1. Your application is calling one of the APIs outside of the context where you do not have access to the `inProgress` state. For more about context see the [MSAL React FAQ](./faq.md)
1. Your application is calling one of the APIs without first checking if interaction is already in progress elsewhere.

The following snippet throws the error when another component has already invoked an interactive API that is in progress:

```javascript
import { useMsal, useIsAuthenticated } from "@azure/msal-react";
import { useEffect } from "react";

export function exampleComponent() {
    const { instance } = useMsal();
    const isAuthenticated = useIsAuthenticated();

    useEffect(() => {
        if (!isAuthenticated) {
            // If another component has already invoked an interactive API this will throw
            await instance.loginPopup();
        }
    }, [isAuthenticated, instance]);
}
```

To fix the previous snippet, check that no other interaction is in progress before invoking `loginPopup`:

```javascript
import { useMsal, useIsAuthenticated } from "@azure/msal-react";
import { InteractionStatus } from "@azure/msal-browser";
import { useEffect } from "react";

export function exampleComponent() {
    const { instance, inProgress } = useMsal();
    const isAuthenticated = useIsAuthenticated();

    useEffect(() => {
        if (!isAuthenticated && inProgress === InteractionStatus.None) {
            await instance.loginPopup();
        }
    }, [isAuthenticated, inProgress, instance]);
}
```

#### Troubleshooting Steps

- [Enable verbose logging](../browser/configuration.md#using-the-config-object) and trace the order of events. Verify that an interactive API is not invoked before another has resolved. If using the redirect flow make sure `handleRedirectPromise` has resolved (done in the `MsalProvider`).

If you are unable to figure out why this error is being thrown please [open an issue](https://github.com/AzureAD/microsoft-authentication-library-for-js/issues/new/choose) and be prepared to share the following information:

- Verbose logs
- A sample app and/or code snippets that we can use to reproduce the issue
- Refresh the page. Does the error go away?
- Open your application in a new tab. Does the error go away?

## See also

There are additional errors that can be thrown by MSAL.js. You can refer to the [full list of errors](../browser/errors.md) covered in MSAL Browser.