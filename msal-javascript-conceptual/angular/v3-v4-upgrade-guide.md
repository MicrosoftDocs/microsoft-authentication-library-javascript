---
title: Upgrade from MSAL Angular v3 to v4
description: Learn how to upgrade your Angular application from MSAL Angular v3 to v4, including security updates and Angular 19 support.
author: Dickson-Mwendia
manager: Dougeby
ms.author: dmwendia
ms.date: 03/06/2026
ms.service: msal
ms.subservice: msal-angular
ms.topic: how-to
ms.reviewer: kengaderdus
#Customer intent: As a developer, I want to upgrade my Angular application from MSAL Angular v3 to v4 so that I can get security updates and Angular 19 support.
---

# Upgrade from MSAL Angular v3 to v4

MSAL Angular v4 includes security updates from MSAL Browser and adds Angular 19 support to the existing Angular 15-18 support.

Please see the [MSAL Browser v3 migration guide](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/v3-migration.md) for browser support and other key changes.

## Changes in `@azure/msal-angular@4`

### Using local storage

Due to changes in MSAL Browser related to local storage encryption, ensure you check that initialization has completed and that interaction status is `None` before calling any account APIs.

```js
this.msalBroadcastService.inProgress$
    .pipe(
        filter(
            (status: InteractionStatus) => status === InteractionStatus.None
        ),
        takeUntil(this._destroying$)
    )
    .subscribe(() => {
        this.loginDisplay = this.authService.instance.getAllAccounts().length > 0;
    });
```

## Samples

The following developer samples are now available:

- [Angular B2C Sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-angular-samples/angular-b2c-sample)
- [Angular Standalone Sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-angular-samples/angular-standalone-sample)

The samples demonstrate basic configuration and usage, and may be improved and added to incrementally.

See [here](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-angular-samples) for a list of the current MSAL Angular samples and the features demonstrated.
