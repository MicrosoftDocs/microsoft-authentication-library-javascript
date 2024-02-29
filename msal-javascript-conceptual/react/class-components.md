---
title: Using MSAL React with class components
description: Learn how to use MSAL React with class components, covering initialization, protecting components, accessing MSAL React context and logging in.
author: EmLauber
manager: CelesteDG

ms.service: msal
ms.subservice: msal-react
ms.topic: conceptual
ms.date: 11/29/2023
ms.author: emilylauber
ms.reviewer: dmwendia,cwerner, owenrichards, kengaderdus
---

# Using MSAL React with class components

MSAL React supports both function components and class components. This article provides guidance on using MSAL React with class components, enabling you to initialize, protect and access MSAL React context in your class components.

It's important to note that you won't be able to use `@azure/msal-react` hooks inside your class components. If you need access to authentication state inside your class component, you'll need to use `@azure/msal-browser` directly to obtain similar functionality.

## Prerequisites



## Initialization

Initialization with class components with MSAL React works very similarly to to function components. Similar to when using function components, you'll need an [`MsalProvider`](/javascript/api/@azure/msal-react/#@azure-msal-react-msalprovider) component at the top level of the component tree that requires access to authentication state.

In the following snippet, the `MsalProvider` component is used to wrap the entire application, making the MSAL instance available to all child components. This enables child components to use MSAL for user authentication, acquiring tokens, and calling protected APIs.

```javascript
import React from "react";
import { MsalProvider } from "@azure/msal-react";
import { PublicClientApplication } from "@azure/msal-browser";

const pca = new PublicClientApplication(config);

class App extends React.Component {
    render() {
        return (
            <MsalProvider instance={pca}>
                <YourAppComponents>
            </ MsalProvider>
        );
    }
}
```

## Protecting Components

In MSAL React, you can protect your components and conditionally render then based on the authentication state of the user. This works similarly to using function components. The main examples are: 

* [`AuthenticatedTemplate`](/javascript/api/@azure/msal-react/#@azure-msal-react-authenticatedtemplate) - This component will only render its children if the user is authenticated. If the user is not authenticated, it will not render anything.
* [`UnauthenticatedTemplate`](/javascript/api/@azure/msal-react/#@azure-msal-react-unauthenticatedtemplate) - This component will only render its children if the user is not authenticated. If the user is authenticated, it will not render anything.
* [`MsalAuthenticationTemplate`](/javascript/api/@azure/msal-react/#@azure-msal-react-msalauthenticationtemplate) - This component attempts to authenticate the user before rendering its children. You can specify the interaction type (redirect or popup) as a prop. If the user is not authenticated, it will initiate the authentication process.

The following snippet shows how to use `AuthenticatedTemplate`, `UnauthenticatedTemplate`, and `MsalAuthenticationTemplate` to protect your React components. Note how `MSAL Provider` wraps around the child components.

```javascript
import React from "react";
import { MsalProvider, AuthenticatedTemplate, UnauthenticatedTemplate, MsalAuthenticationTemplate } from "@azure/msal-react";
import { PublicClientApplication, InteractionType } from "@azure/msal-browser";

const pca = new PublicClientApplication(config);

class App extends React.Component {
    render() {
        return (
            <MsalProvider instance={pca}>
                <AuthenticatedTemplate>
                    <span>This will only render for authenticated users</span>
                </ AuthenticatedTemplate>
                <UnauthenticatedTemplate>
                    <span>This will only render for unauthenticated users</span>
                </ UnauthenticatedTemplate>
                <MsalAuthenticationTemplate interactionType={InteractionType.Popup}>
                    <span>This will only render for authenticated users.</span>
                </ MsalAuthenticationTemplate>
            </ MsalProvider>
        );
    }
}
```

## Accessing MSAL React context in a class component

The `useMsal` hook can't be used to access the MSAL React context in a class component. This is because hooks can only be used in function components where you can use features without an instance. Because class components have instances, you have 2 other options. You can either use the raw context directly or you can use the `withMsal` higher order component to inject the context into your component's props. <!--IMsalContext?-->

### Accessing raw context

The following snippet shows how to use the `MsalContext` to access the raw context in a class component.

```javascript
import React from "react";
import { MsalProvider, MsalContext } from "@azure/msal-react";
import { PublicClientApplication } from "@azure/msal-browser";

const pca = new PublicClientApplication(config);

class App extends React.Component {
    render() {
        return (
            <MsalProvider instance={pca}>
                <YourClassComponent/>
            </ MsalProvider>
        );
    }
}

class YourClassComponent extends React.Component {
    static contextType = MsalContext;

    render() {
        const isAuthenticated = this.context.accounts.length > 0;
        if (isAuthenticated) {
            return <span>There are currently {this.context.accounts.length} users signed in!</span>
        }
    }
}
```

For a working example, refer to [*ProfileRawContext.jsx*](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-react-samples/react-router-sample/src/pages/ProfileRawContext.jsx) in our [react-router-sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-react-samples/react-router-sample).

### Accessing via withMsal HOC

The alternative is to use the `withMsal` higher order component to inject the context into your component's props. The following snippet shows how to use the `withMsal` HOC to access the MSAL React context in a class component.

```javascript
import React from "react";
import { MsalProvider, withMsal } from "@azure/msal-react";
import { PublicClientApplication } from "@azure/msal-browser";

const pca = new PublicClientApplication(config);

class YourClassComponent extends React.Component {
    render() {
        const isAuthenticated = this.props.msalContext.accounts.length > 0;
        if (isAuthenticated) {
            return <span>There are currently {this.props.msalContext.accounts.length} users signed in!</span>
        }
    }
}

const YourWrappedComponent = withMsal(YourClassComponent);

class App extends React.Component {
    render() {
        return (
            <MsalProvider instance={pca}>
                <YourWrappedComponent />
            </ MsalProvider>
        );
    }
}
```

For a working example, refer to [*ProfileWithMsal.jsx*](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-react-samples/react-router-sample/src/pages/ProfileWithMsal.jsx) in [react-router-sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-react-samples/react-router-sample).

## Logging in using a class component

Regardless of which approach you take to get the `MSAL React` context, the usage will be the same. Once you have the context object you can invoke [any of the APIs](/javascript/api/@azure/msal-browser/publicclientapplication) on `PublicClientApplication`, inspect the accounts signed in, or determine if authentication is currently in progress.

The following examples will show how to login using the `withMsal` HOC approach but you can quickly adapt to the other approach if needed.

> [!NOTE]
> It's important to remember that an `MsalProvider` component must be rendered at any level above any component that uses context. The following examples assume there is a provider and won't demonstrate this.

### Logging in as a result of clicking a button

The following snippet defines a React class component `LoginButton` that uses the `withMsal` higher-order component. It renders a button that either triggers a login popup if the user is not authenticated, or logs out the user if they are authenticated.

```javascript
import React from "react";
import { withMsal } from "@azure/msal-react";

class LoginButton extends React.Component {
    render() {
        const isAuthenticated = this.props.msalContext.accounts.length > 0;
        const msalInstance = this.props.msalContext.instance;
        if (isAuthenticated) {
            return <button onClick={() => msalInstance.logout()}>Logout</button>    
        } else {
            return <button onClick={() => msalInstance.loginPopup()}>Login</button>
        }
    }
}

export default YourWrappedComponent = withMsal(LoginButton);
```

### Logging in on page load

The following snippet defines a React class component `ProtectedComponent` that uses the `withMsal` higher-order component. It attempts to authenticate the user upon mounting and updating, and displays whether the user is authenticated or if authentication is in progress on the loading page.

```javascript
import React from "react";
import { withMsal } from "@azure/msal-react";
import { InteractionStatus } from "@azure/msal-browser";

class ProtectedComponent extends React.Component {
    callLogin() {
        const isAuthenticated = this.props.msalContext.accounts.length > 0;
        const msalInstance = this.props.msalContext.instance;

        // If a user is not logged in and authentication is not already in progress, invoke login
        if (!isAuthenticated && this.props.msalContext.inProgress === InteractionStatus.None) {
            msalInstance.loginPopup();
        }
    }
    componentDidMount() {
        this.callLogin();
    }

    componentDidUpdate() {
        this.callLogin();
    }
    
    render() {
        const isAuthenticated = this.props.msalContext.accounts.length > 0;
        if (isAuthenticated) {
            return <span>User is authenticated</span>
        } else {
            return <span>Authentication in progress</span>;
        }
    }
}

export default YourWrappedComponent = withMsal(ProtectedComponent);
```

## See also

- Check out our [react-router-sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-react-samples/react-router-sample).