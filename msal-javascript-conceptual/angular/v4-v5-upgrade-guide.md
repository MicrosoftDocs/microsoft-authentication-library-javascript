---
title: Upgrade from MSAL Angular v4 to v5
description: Learn how to upgrade your Angular application from MSAL Angular v4 to v5, including breaking changes and migration guidance.
author: Dickson-Mwendia
manager: Dougeby
ms.author: dmwendia
ms.date: 03/06/2026
ms.service: msal
ms.subservice: msal-angular
ms.topic: how-to
ms.reviewer: cwerner, owenrichards, kengaderdus
#Customer intent: As a developer, I want to upgrade my Angular application from MSAL Angular v4 to v5 so that I can use the latest features and Angular 19 support.
---

# Upgrade from MSAL Angular v4 to v5

MSAL Angular v5 requires a minimum version of Angular 19 and drops support for Angular 15, 16, 17, and 18.

Please see the [MSAL Browser v4 to v5 migration guide](../browser/v4-migration.md) for browser support and other key changes in the underlying `@azure/msal-browser` library.

## Breaking changes in `@azure/msal-angular@5`

### Strict matching for `protectedResourceMap`

In msal-angular v5, URL pattern matching for `protectedResourceMap` entries uses strict matching semantics by default. Strict matching treats pattern metacharacters as literals, anchors matches to the full URL component, and applies host wildcard rules that do not span dot separators. If your v4 configuration relied on looser matching behavior, update your `protectedResourceMap` patterns to align with strict matching, or set `strictMatching` to `false` to retain legacy behavior temporarily. See [MSAL Interceptor docs](./msal-interceptor.md#strict-matching-strictmatching) for more details.

### `logout()` removed

`logout()` has been removed. Use `logoutRedirect()` or `logoutPopup()` instead.

```typescript
// BEFORE (v4)
this.authService.logout();

// AFTER (v5)
this.authService.logoutRedirect();
// or
this.authService.logoutPopup();
```

## Other changes in `@azure/msal-angular@5`

### `inject(TOKEN)` syntax

`MSAL_INSTANCE`, `MSAL_GUARD_CONFIG`, `MSAL_INTERCEPTOR_CONFIG`, and `MSAL_BROADCAST_CONFIG` now resolve to types instead of strings in order to support `inject(TOKEN)` syntax. This change may cause TypeScript errors in applications without explicit typing.

### `handleRedirectObservable()` options

`handleRedirectObservable()` now accepts an optional `HandleRedirectPromiseOptions` object, which includes the `navigateToLoginRequestUrl` option that was moved from the configuration in `@azure/msal-browser@5`. See the [redirects documentation](./redirects.md#handleredirectobservable-options) for more details.

```typescript
// BEFORE (msal-browser v4 configuration)
const msalConfig = {
  auth: {
    clientId: 'your-client-id',
    navigateToLoginRequestUrl: false // This option has moved
  }
};

// AFTER (msal-angular v5)
this.authService.handleRedirectObservable({
  navigateToLoginRequestUrl: false
}).subscribe();
```

> [!NOTE]
> Passing a hash string directly to `handleRedirectObservable(hash)` is deprecated. Use the options object instead: `handleRedirectObservable({ hash: "#..." })`.

## Samples

The following developer samples are now available:

- [Angular B2C Sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-angular-samples/angular-b2c-sample)
- [Angular Standalone Sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-angular-samples/angular-standalone-sample)

See [here](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-angular-samples) for a list of the current MSAL Angular samples and the features demonstrated.
