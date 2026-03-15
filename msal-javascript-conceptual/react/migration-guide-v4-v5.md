---
title: Migrate from MSAL React v3 to v5
description: Learn how to migrate your React application from MSAL React v3 to v5, including React 19 requirements, CRA to Vite migration, and InteractionStatus changes.
author: Dickson-Mwendia
manager: Dougeby
ms.author: dmwendia
ms.date: 03/15/2026
ms.service: msal
ms.subservice: msal-react
ms.topic: how-to
ms.reviewer: kengaderdus
#Customer intent: As a developer upgrading from MSAL React v3 to v5, I want a migration guide so I can update my React app to work with React 19 and the latest MSAL packages.
---

# Migrate from MSAL React v3 to v5

> [!NOTE]
> There's no MSAL React v4 release. The package version was incremented from v3 directly to v5 to align `msal-react` versioning with the other MSAL.js libraries. No separate v4 feature set exists.

See the [MSAL Browser v4-v5 migration guide](../browser/v4-migration.md) for browser support and other key changes.

## Migration paths

- **v3 -> v5**: Follow this guide, then apply the [MSAL Browser v4-v5 migration guide](../browser/v4-migration.md), especially the redirect bridge setup.
- **v1/v2 -> v5**: The v1 -> v2 and v2 -> v3 updates were peer dependency version updates only for most apps. Move to v3 first, then follow the v3 -> v5 guidance in this document plus redirect bridge setup.

## Redirect bridge setup (required)

MSAL Browser v5 requires a dedicated redirect page/bridge for authentication flows.

See the [COOP section in the MSAL Browser v4-v5 migration guide](../browser/v4-migration.md#cross-origin-opener-policy-coop-support).

## Dropped support for old React versions

MSAL React v5 supports React 19.2.1 or greater. It no longer supports React 16, 17, or 18.

## React 18 compatibility note

> [!WARNING]
> React 18 has reached end of life, which is why MSAL React v5 dropped support for it.
>
> If your app is still on React 18, installing `@azure/msal-react@^5` might fail due to peer dependency constraints.
>
> - Temporary install workaround: `npm install --legacy-peer-deps`
> - This might allow installation and basic flows may continue to work in some apps, but React 18 isn't supported or validated for v5
> - You might see untested behavior around rendering/lifecycle timing, StrictMode interactions, or future patch updates
>
> For production workloads, upgrade React to 19.2.1 or greater before moving to `@azure/msal-react@^5`.

## Migrating from Create React App (react-scripts)

Create React App is deprecated and `react-scripts` doesn't support React 19. If your app uses `react-scripts`, you **must** migrate to a different build tool before upgrading to `@azure/msal-react@^5`.

**Recommended: Migrate to [Vite](https://vite.dev/)**

1. Remove `react-scripts` and install Vite + the React plugin:
    ```bash
    npm uninstall react-scripts
    npm install --save-dev vite @vitejs/plugin-react
    ```

2. Add `"type": "module"` to `package.json`.

3. Move `public/index.html` to the project root as `index.html`, replace `%PUBLIC_URL%/` references with `/`, and add the entry point script tag:
    ```html
    <!-- Before (CRA): no script tag needed, CRA injects it -->
    <!-- After (Vite): add before </body> -->
    <script type="module" src="/src/index.jsx"></script>
    ```

4. Create a `vite.config.js` at project root:
    ```js
    import { defineConfig } from "vite";
    import react from "@vitejs/plugin-react";
    import { resolve } from "path";

    export default defineConfig({
        plugins: [react()],
        server: {
            port: 3000,
        },
        build: {
            rollupOptions: {
                input: {
                    main: resolve(__dirname, "index.html"),
                    redirect: resolve(__dirname, "public/redirect.html"),
                },
            },
        },
    });
    ```

5. Update `package.json` scripts:
    ```json
    "scripts": {
        "start": "vite",
        "build": "vite build",
        "preview": "vite preview"
    }
    ```

6. Replace `process.env.REACT_APP_*` references with `import.meta.env.VITE_*` and rename environment variables accordingly (for example, `REACT_APP_CLIENT_ID` → `VITE_CLIENT_ID`).

7. Remove CRA-specific environment variables (`SKIP_PREFLIGHT_CHECK`, `DISABLE_ESLINT_PLUGIN`) from `.env` files.

8. Upgrade React and install MSAL:
    ```bash
    npm install react@^19.2.1 react-dom@^19.2.1
    npm install @azure/msal-browser@^5 @azure/msal-react@^5
    ```

9. Set up the [redirect bridge](../browser/redirect-bridge.md#vite) for your Vite app.

> **Samples:** See the [react-router-sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-react-samples/react-router-sample), [typescript-sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-react-samples/typescript-sample), and [b2c-sample](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-react-samples/b2c-sample) for complete Vite-based examples.

## Correct logout bug

MSAL React v5 has fixed a bug affecting the `useMsalAuthentication` hook and `MsalAuthenticationTemplate`. Logging out now clears all state associated with the user.

## `InteractionStatus` changes

For `InteractionStatus`: `Login`, `SsoSilent`, and `AcquireToken` are now consolidated into `AcquireToken`.

### Migration example

If your app previously checked multiple in-progress statuses, simplify to the consolidated `AcquireToken` status.

```ts
import { InteractionStatus } from "@azure/msal-browser";

// Before (v3-style checks)
const tokenInteractionInProgress =
	inProgress === InteractionStatus.Login ||
	inProgress === InteractionStatus.SsoSilent ||
	inProgress === InteractionStatus.AcquireToken;

// After (v5)
const tokenInteractionInProgress =
	inProgress === InteractionStatus.AcquireToken;

if (!tokenInteractionInProgress) {
	// safe to initiate a new auth/token request
}
```
