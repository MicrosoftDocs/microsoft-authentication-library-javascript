---
title: Logging in MSAL Angular
description: Learn about logging in MSAL Angular
author: Dickson-Mwendia
manager: CelesteDG

ms.service: msal
ms.subservice: msal-angular
ms.topic: how-to
ms.date: 05/21/2025
ms.author: dmwendia
ms.reviewer: cwerner, owenrichards, kengaderdus
---
# Logging in MSAL Angular

The logger definition has the following properties:

1. correlationId
1. logLevel
    * logLevels include: `Error`, `Warning`, `Info`, `Trace`, and `Verbose`
1. piiLoggingEnabled

You can enable logging in your app as shown below:

```js
import { LogLevel, PublicClientApplication } from '@azure/msal-browser';

export function loggerCallback(logLevel, message) {
    console.log(message);
}

@NgModule({
    imports: [ 
        MsalModule.forRoot(new PublicClientApplication({
            auth: {
                clientId: 'Your client ID',
            },
            system: {
                loggerOptions: {
                    loggerCallback,
                    piiLoggingEnabled: true,
                    logLevel: LogLevel.Info
                }
            }
        }))
    ]
})
```

The `logger` can also be set dynamically by using `MsalService.setLogger()`.

```js
this.authService.setLogger(new Logger({
    loggerCallback: (logLevel, message, piiEnabled) => {
        console.log('MSAL Logging: ', message);
    },
    piiLoggingEnabled: false,
    logLevel: LogLevel.Info
}));
```
