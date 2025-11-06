---
title: Version Checking
excerpt: ''
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
# Module: `modules.version` (App version gating & update UX)

## High-Level Summary

`modules.version` centralizes **minimum-version checks** for a hybrid (Cordova/PhoneGap) app. It parses semantic versions (`major.minor.build`), compares the **currently running app version** against a **required minimum**, and—if outdated—presents an **update alert** with deep links to the App Store / Google Play and guards app load. It also exposes helpers to test comparisons, fetch the current required version for the active device, and trigger the in-store update flow.

***

## Public API

| Method              | Signature                                                                 | Returns        | Description                                                                                                                                                                                                                                                                                                                               |
| ------------------- | ------------------------------------------------------------------------- | -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `min`               | `min(ver: string, test?: string): boolean`                                | `true/false`   | Compares `ver` (minimum required) to the **running app version** (from `window.app_conf.version`, or `test` if provided). Returns `true` if the running app is **≥** `ver`, else `false`. No-op `true` for non-PhoneGap/web contexts.                                                                                                     |
| `test`              | `test(): void`                                                            | —              | Runs a set of canned comparisons and logs PASS/FAIL to console.                                                                                                                                                                                                                                                                           |
| `getCurrentVersion` | `getCurrentVersion(): string`                                             | version string | Returns the **required min** version for the current device from `modules.version.current[app.device]`.                                                                                                                                                                                                                                   |
| `updateAlert`       | `updateAlert(): void`                                                     | —              | Shows an in-app modal prompting the user to update (uses the `$.fn.alert` plugin and a `template: 'updatealert'`).                                                                                                                                                                                                                        |
| `updateInAppStore`  | `updateInAppStore(): void`                                                | —              | Opens the platform store page (iOS App Store / Google Play) using Cordova InAppBrowser when available; falls back to https links. Shows a spinner and wires a `resume` handler to clear it.                                                                                                                                               |
| `check`             | `check(current: Record<Device,string>, disable_alert?: boolean): boolean` | `true/false`   | Main entry: set `modules.version.current = current`; if running in PhoneGap and the app’s version is **below** the required min for `app.device`, it **shows an update modal** (unless `disable_alert`), registers a `resume` listener to clear spinner, and returns `false` to **block load**. Returns `true` when OK or not applicable. |

***

## Inputs / Options

Although not option-based, two inputs are significant:

| Input                  | Type                    | Required | Example                              | Notes                                                                                     |
| ---------------------- | ----------------------- | -------: | ------------------------------------ | ----------------------------------------------------------------------------------------- |
| `current` (to `check`) | `Record<Device,string>` |      Yes | `{ iOS: "4.8.1", Android: "4.8.0" }` | Map of **minimum required versions** per device. `app.device` selects the key at runtime. |
| `ver` (to `min`)       | `string`                |      Yes | `"4.8.1"`                            | Minimum required version in `MAJOR.MINOR.BUILD` format (3 numeric parts).                 |
| `test` (to `min`)      | `string`                |       No | `"4.8.0"`                            | Overrides `window.app_conf.version` for comparison (useful in unit tests).                |

***

## Dependencies & Side Effects

- **Environment checks:** `isPhoneGap()`; uses `window.app_conf.version` and `window.app_conf.app_identifier`.
- **Globals used:** `app.device`, `app.name`, `app.isdev`, `app.ios_store_id`.
- **UI/Plugins:**

  - `$('body').alert(...)` (the alert/modal plugin) with `template: 'updatealert'`.
  - `$('body').spin(...)` spinner plugin.
  - `phone.statusBar.set()` (status bar reset).
- **Linking:** `cordova.InAppBrowser` if available; otherwise `_.openLink` with https/market/itms-apps URLs.
- **Side effects:**

  - `min()` sets `app.appversion = { major, minor, build }` (parsed from the running app version).
  - `check()` registers a `document.addEventListener('resume', ...)` that clears spinners.

***

## Behavior Details

### Version Comparison (`min`)

- Gate condition: runs only when `(isPhoneGap() && window.app_conf && window.app_conf.version) || test` is truthy. Otherwise returns `true` (i.e., no restriction in web environments).
- Parses both `ver` (required minimum) and the **current app version** (`window.app_conf.version` or `test`) into `{major, minor, build}` and compares lexicographically:

  - If required **major** > current major → **false** (too old).
  - If majors equal, compare **minor** similarly.
  - If minors equal, compare **build** similarly.
  - Otherwise **true** (meets or exceeds min).

### Update UX (`updateAlert` & `updateInAppStore`)

- Renders an overlay with a single button:

  - iOS: “Update from App Store”
  - Android: “Update From Google Play”
- Button tap opens the relevant store page. While navigating, a full-screen spinner shows; on app `resume`, spinner hides, alert closes, and status bar resets.

### Gate on App Start (`check`)

- If device is PhoneGap and a current app version is present:

  - Retrieves `min = current[app.device]`.
  - If `min` exists and `modules.version.min(min)` is **false**:

    - When `disable_alert` is falsy → show `updateAlert()`, wire `resume` → **return false** to block further load.
    - When `disable_alert` is truthy → **return false** silently (caller can handle UX).
- Returns `true` if no min is set, environment is non-PhoneGap, or version is adequate.

***

## Usage Example

```js
// 1) On app bootstrap, enforce minimum versions per platform
modules.version.check({
  iOS:     '4.8.1',
  Android: '4.8.0'
});
// If returns false, caller can choose to halt further initialization.
```

```js
// 2) Ad-hoc comparison (e.g., feature gating) — pass a test current version in web/dev
if (!modules.version.min('5.0.0', '4.9.3')) {
  console.warn('Feature requires at least 5.0.0');
}
```

```js
// 3) Manually present update prompt (e.g., from a settings screen)
modules.version.updateAlert();
```

***

## Notes & Edge Cases (rigorous checks)

1. **Version format assumptions**  
   Both `ver` and current (`window.app_conf.version` or `test`) are split on `'.'` and `parseInt` is used. **Non-numeric or missing parts** will produce `NaN`, making comparisons unreliable.  
   **Recommendation:** Validate and coerce missing parts to `0`; bail gracefully on malformed versions.

2. **Operator precedence in the gate**  
   `if (isPhoneGap() && window.app_conf && window.app_conf.version || test)` is evaluated as `(A && B && C) || test`. This is likely intended (allow `test` to force-run), but be aware `test` **always** enables comparison even in web. Good for unit tests.

3. **Side effect: sets `app.appversion`**  
   `min()` writes `app.appversion` on every call. If other code depends on this field, that’s fine; otherwise it’s a hidden side effect.  
   **Recommendation:** Document or confine this mutation.

4. **Device key alignment**  
   `getCurrentVersion()` and `check()` use `app.device` to index `current`. Ensure your keys match (`'iOS'` vs `'iOS'`, `'Android'` vs `'Android'`) exactly.

5. **UI dependencies**  
   `updateAlert()` assumes the `updatealert` template exists for the alert plugin and that the spinner + statusBar APIs are available.  
   **Recommendation:** Guard with feature detection or provide graceful fallbacks.

6. **Store URLs**

   - iOS: uses both `itms-apps://itunes.apple.com/app/id{store_id}` (in-app) and `https://itunes.apple.com/...` (fallback).
   - Android: uses `market://details?id={app_identifier}` or Play Store https link.  
     Ensure `app.ios_store_id` and `window.app_conf.app_identifier` are set.

7. **Blocking load semantics**  
   `check()` returning `false` is the mechanism to halt the app. Ensure the caller checks the boolean and **early-returns** from initialization code.

8. **Dev message branch**  
   There is a dev-only `_alert(...)` path commented/guarded by `app.isdev && false`. It currently never fires; delete or wire appropriately.

***