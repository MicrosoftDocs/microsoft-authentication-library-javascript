---
title: Initializing MSAL Angular
description: Learn how to initialize MSAL in your Angular application
author: Dickson-Mwendia
manager: Doueby

ms.service: msal
ms.subservice: msal-angular
ms.topic: concept-article
ms.date: 05/21/2025
ms.author: dmwendia
ms.reviewer: cwerner, owenrichards, kengaderdus
---
# Initializing MSAL angular

Before using `@azure/msal-angular`, [register an application in Microsoft Entra ID](/entra/identity-platform/quickstart-register-app) to get the `clientId`.

## Include and initialize the MSAL module in your app module

Import `MsalModule` into app.module.ts. To initialize the MSAL module, pass the clientId of your application, which you get from the application registration.

```js
import { NgModule } from '@angular/core';
import { HTTP_INTERCEPTORS, HttpClientModule } from '@angular/common/http';
import { AppComponent } from './app.component';
import { MsalModule, MsalService, MsalGuard, MsalInterceptor, MsalBroadcastService, MsalRedirectComponent } from "@azure/msal-angular";
import { PublicClientApplication, InteractionType, BrowserCacheLocation } from "@azure/msal-browser";

@NgModule({
    imports: [
        MsalModule.forRoot( new PublicClientApplication({ // MSAL Configuration
            auth: {
                clientId: "Your client ID",
                authority: "Your authority",
                redirectUri: "Your redirect Uri",
            },
            cache: {
                cacheLocation : BrowserCacheLocation.LocalStorage,
                storeAuthStateInCookie: true, // Deprecated, removed in a future version
            },
            system: {
                loggerOptions: {
                    loggerCallback: () => {},
                    piiLoggingEnabled: false
                }
            }
        }), {
            interactionType: InteractionType.Redirect, // MSAL Guard Configuration
        }, {
            interactionType: InteractionType.Redirect, // MSAL Interceptor Configuration
        })
    ],
    providers: [
        {
            provide: HTTP_INTERCEPTORS,
            useClass: MsalInterceptor,
            multi: true
        },
        MsalService,
        MsalGuard,
        MsalBroadcastService
    ],
    bootstrap: [AppComponent, MsalRedirectComponent]
})
export class AppModule {}
```

## Secure the routes in your application

Add authentication to secure specific routes in your application by adding `canActivate: [MsalGuard]` to your route definition. Add it to parent or child routes. When a user visits these routes, the library prompts the user to authenticate.

Learn more in our [`MsalGuard` doc](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/docs/msal-guard.md) about configuration and considerations, including using additional interfaces.

Here is an example of a route defined with the `MsalGuard`:

```js
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { ProfileComponent } from './profile/profile.component';
import { MsalGuard } from '@azure/msal-angular';

const routes: Routes = [
    {
        path: 'profile',
        component: ProfileComponent,
        canActivate: [MsalGuard]
    },
    {
        path: '',
        component: HomeComponent
    },
];

@NgModule({
    imports: [RouterModule.forRoot(routes)],
    exports: [RouterModule]
})
export class AppRoutingModule { }
```

## Get tokens for web API calls

`@azure/msal-angular` allows you to add an Http interceptor (`MsalInterceptor`) in your `app.module.ts` as follows. The `MsalInterceptor` will obtain tokens and add them to all your Http requests in API calls based on the `protectedResourceMap`. See our [MsalInterceptor doc](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/docs/msal-interceptor.md) for more details on configuration and use.

```js
import { NgModule } from '@angular/core';
import { HTTP_INTERCEPTORS } from '@angular/common/http';
import { AppComponent } from './app.component';
import { MsalModule, MsalService, MsalGuard, MsalInterceptor, MsalBroadcastService, MsalRedirectComponent } from "@azure/msal-angular";
import { PublicClientApplication, InteractionType, BrowserCacheLocation } from "@azure/msal-browser";

@NgModule({
    imports: [
        MsalModule.forRoot( new PublicClientApplication({ // MSAL Configuration
            auth: {
                clientId: "Your client ID",
                authority: "Your authority",
                redirectUri: "Your redirect Uri",
            },
            cache: {
                cacheLocation : BrowserCacheLocation.LocalStorage,
                storeAuthStateInCookie: true, // Deprecated, will be removed in future version
            },
            system: {
                loggerOptions: {
                    loggerCallback: () => {},
                    piiLoggingEnabled: false
                }
            }
        }), {
            interactionType: InteractionType.Redirect, // MSAL Guard Configuration
        }, {
            interactionType: InteractionType.Redirect, // MSAL Interceptor Configuration
            protectedResourceMap: new Map([
                ['https://graph.microsoft.com/v1.0/me', ['user.read']],
                ['https://api.myapplication.com/users/*', ['customscope.read']],
                ['http://localhost:4200/about/', null] 
            ])
        })
    ],
    providers: [
        {
            provide: HTTP_INTERCEPTORS,
            useClass: MsalInterceptor,
            multi: true
        },
        MsalService,
        MsalGuard,
        MsalBroadcastService
    ],
    bootstrap: [AppComponent, MsalRedirectComponent]
})
export class AppModule {}
```

Using the `MsalInterceptor` is optional. You might want to explicitly get tokens using the acquireToken APIs instead.

Note that the `MsalInterceptor` is provided for your convenience and might not fit all use cases. Write your own interceptor if you have specific needs that aren't addressed by the `MsalInterceptor`. 

## Subscribe to events

MSAL provides an event system that emits events related to authentication and MSAL. To use events, add the `MsalBroadcastService` to the constructor in your component or service. 

### 1. How to subscribe to events

```js
import { EventMessage, EventType } from '@azure/msal-browser';
import { filter } from 'rxjs/operators';

this.msalBroadcastService.msalSubject$
    .pipe(
        filter((msg: EventMessage) => msg.eventType === EventType.LOGIN_SUCCESS)
    )
    .subscribe((result) => {
        // do something here
    });
```

### 2. Available events

Find the list of events available to MSAL in the [`@azure/msal-browser` event documentation.](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/events.md)

### 3. Unsubscribing

Unsubscribing is important. Implement `ngOnDestroy()` in your component to unsubscribe.

```js
import { EventMessage, EventType } from '@azure/msal-browser';
import { filter, Subject, takeUntil } from 'rxjs';

private readonly _destroying$ = new Subject<void>();

this.msalBroadcastService.msalSubject$
    .pipe(
        filter((msg: EventMessage) => msg.eventType === EventType.LOGIN_SUCCESS),
        takeUntil(this._destroying$)
    )
    .subscribe((result) => {
        this.checkAccount();
    });

ngOnDestroy(): void {
    this._destroying$.next(null);
    this._destroying$.complete();
}
```

## Next steps

You're ready to use `@azure/msal-angular` [public APIs](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/docs/public-apis.md).