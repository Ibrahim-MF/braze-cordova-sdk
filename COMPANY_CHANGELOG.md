# Company Changelog

All company-specific changes on top of upstream Braze Cordova SDK.
Upstream version changes are recorded in [UPSTREAM.md](UPSTREAM.md).

---

## [17.0.0-company.1] — 2026-09-05

Base: upstream `17.0.0`

### Added

- **`www/BrazePlugin.js`** — `BrazePlugin.prototype.initialize(options, successCallback, errorCallback)`:
  Deferred country-aware initialization entry point. Passes a two-letter country code (`EG` / `MA`)
  to the native layer. The native layer resolves the API key; no key is ever exposed to JavaScript.

- **`src/android/BrazePlugin.kt`** — `"initialize"` execute action:
  Resolves the country-specific Android API key from `com.company.braze.android_api_key.<COUNTRY>`
  preferences, then calls `configureFromCordovaPreferences()`. Double-init is prevented via the
  `companyBrazeInitialized` flag. All other Braze actions are rejected until init completes.

- **`src/android/BrazePlugin.kt`** — `resolveCompanyAndroidApiKey(country)` private method:
  Resolution order: `com.company.braze.android_api_key.<COUNTRY>` →
  `com.braze.android_api_key` → `com.braze.api_key` (deprecated fallback).

- **`src/android/BrazePlugin.kt`** — `COMPANY_ANDROID_API_KEY_PREFIX` / `COMPANY_SUPPORTED_COUNTRIES`
  constants in `companion object`.

- **`src/android/BrazePlugin.kt`** — `companyBrazeInitialized` instance field:
  Guards all four lifecycle methods (`onStart`, `onStop`, `onResume`, `onPause`) so session and
  in-app message management does not run before a country key is resolved.

- **`src/ios/BrazePlugin.m`** — `initializeBraze:` CDV command handler:
  Country-aware deferred initialization for iOS. Resolves `com.company.braze.ios_api_key.<COUNTRY>`,
  falls back to the key read by `pluginInitialize`, then delegates to the existing
  `didFinishLaunchingListener:` method.

### Changed

- **`src/android/BrazePlugin.kt`** — `pluginInitialize()`:
  Removed `configureFromCordovaPreferences(preferences)` call. Braze is no longer configured at
  plugin load; configuration is deferred to the `"initialize"` execute action.

- **`src/android/BrazePlugin.kt`** — `onStart()` / `onStop()` / `onResume()` / `onPause()`:
  All session and in-app message manager calls are now guarded by `companyBrazeInitialized`.

- **`src/ios/BrazePlugin.m`** — `pluginInitialize`:
  Removed the `UIApplicationDidFinishLaunchingNotification` observer that previously triggered
  automatic Braze initialization at app launch. Initialization is now deferred.

- **`src/ios/BrazePlugin.m`** — `didFinishLaunchingListener:`:
  Added a `[BrazePlugin braze] != nil` guard at the top to prevent double initialization.

### Rationale

MoneyFellows operates in Egypt (EG) and Morocco (MA) with separate Braze workspaces and API keys.
The single-key upstream plugin cannot select the correct workspace before the user's country is
known. Deferred initialization lets the app resolve the country at runtime and pass it to the native
layer, which then picks the right key from config — without ever exposing a real key to JavaScript.
