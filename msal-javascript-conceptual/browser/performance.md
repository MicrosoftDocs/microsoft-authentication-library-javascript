---
title: Measuring performance in MSAL.js 
description: Learn more about telemetry and performance of token acquisition in MSAL.js
author: Dickson-Mwendia
manager: Dougeby
ms.service: msal
ms.subservice: msal-js
ms.topic: concept-article
ms.date: 05/21/2025
ms.author: dmwendia
ms.reviewer: cwerner, owenrichards, kengaderdus
---

# Measuring performance

Applications that want to measure the performance of authentication flows in MSAL.js can do so manually, or consume the performance measures taken by the library itself.
Consuming performance measurements requires setting performance client in [telemetry configuration options](./configuration.md#telemetry-config-options) and adding performance callback.

### Set telemetry performance client

```javascript
import { PublicClientApplication, BrowserPerformanceClient } from "@azure/msal-browser";

const msalConfig = {
    auth: {
        ...
    },
    cache: {
        ...
    },
    system: {
        ...
    }
}

const msalInstance = new PublicClientApplication({
    ...msalConfig,
    telemetry: {
        client: new BrowserPerformanceClient(msalConfig)
    }
});
msalInstance.initialize();
```

### Add performance callback

Applications can register a callback to receive performance measurements taken by the library. These measurement will include end-to-end measurements for top-level APIs, as well as measurements for important internal APIs.

**Note for MSFT first-party applications**: We will be publishing an internal build of `@azure/msal-browser` that is already instrumented to capture this telemetry. Contact us for more details.

#### Example

```typescript
const msalInstance = new PublicClientApplication(config);

msalInstance.addPerformanceCallback((events: PerformanceEvent[]) => {
    events.forEach(event => {
        console.log(event);
    });
});
```

Example event:

```typescript
const event: PerformanceEvent = {
    correlationId: "aaaa0000-bb11-2222-33cc-444444dddddd",
	durationMs: 1873,
	endPageVisibility: "hidden",
	fromCache: false,
	name: "acquireTokenSilent",
	startPageVisibility: "visible",
	startTimeMs: 1636414041888,
	success: true,
    silentCacheClientAcquireTokenDurationMs: 0,
    silentRefreshClientAcquireTokenDurationMs: 150,
    silentIframeClientAcquireTokenDurationMs: 0
    cryptoOptsGetPublicKeyThumbprintDurationMs: 200,
    cryptoOptsSignJwtDurationMs: 8,
    clientId: "00001111-aaaa-2222-bbbb-3333cccc4444",
    authority: "https://login.microsoftonline.com/common",
    libraryName: "@azure/msal-browser-1p",
    libraryVersion: "2.22.2-beta.2",
    appName: "my-application",
    appVersion: "1.0.0"
}
```

The complete details for `PerformanceEvents` objects can be found [here](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/src/telemetry/performance/PerformanceClient.ts). Below is a list of some notable properties:

| **Property**                       | Type      | Description                                                            |
| ---------------------------------- | --------- | ---------------------------------------------------------------------- |
| `name`                             | `string`  | Name of the operation, usually matches the top-level API name (e.g. `acquireTokenSilent`, `acquireTokenByCode`, `ssoSilent`). |
| `durationMs`                       | `number`  | End-to-end duration in milliseconds for the operation.                 |
| `success`                          | `boolean` | Whether the operation was successful or not.                           |
| `fromCache`                        | `boolean` | Whether the operation retrieved the result from the cache.             |
| `correlationId`                    | `string`  | Correlation ID used for the operation (preferably unique per request). |
| `libraryVersion`                   | `string`  | Version of MSAL.js used for the operation.                             |
| `authority`                        | `string`  | Authority used for the operation.                                      |
| `<internalFunctionName>DurationMs` | `number`  | Duration in milliseconds for an internal operation.                    |

### removePerformanceCallback

The `addPerformanceCallback` API will return a callback id, which an application can pass to `PublicClientApplication.removePerformanceCallback` to unregister that callback from receiving performance events. It will return a boolean indicating whether or not the callback was successfully removed.

#### Example

```typescript
const msalInstance = new PublicClientApplication(config);

const callbackId: string = msalInstance.addPerformanceCallback((events: PerformanceEvent[]) => {
    events.forEach(event => {
        console.log(event);
    });
});

const removed: boolean = msalInstance.removePerformanceCallback(callbackId);
```
