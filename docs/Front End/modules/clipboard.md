---
title: Clipboard
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
# Module: `modules.clipboard` (Cross-platform copy-to-clipboard)

## High-Level Summary

`modules.clipboard` provides a single method, `copy`, that copies arbitrary text to the system clipboard across **Cordova/PhoneGap (iOS)** and **modern web browsers**. On iOS Cordova it uses the native clipboard plugin; on the web it uses the **Async Clipboard API** with both `text/html` and `text/plain` flavors for rich-text compatibility. It accepts optional success/failure callbacks and attempts to offer graceful fallbacks (commented legacy path using ClipboardJS).

***

## API

### `copy(text, success?, fail?, force?)`

| Param     | Type       | Required | Description                                                                                            |
| --------- | ---------- | -------: | ------------------------------------------------------------------------------------------------------ |
| `text`    | `string`   |  **Yes** | The content to copy. Written as both HTML and plain text in web mode.                                  |
| `success` | `function` |       No | Invoked after a successful copy.                                                                       |
| `fail`    | `function` |       No | Invoked on failure (Cordova path uses it; web path currently uses `alert('error')` instead—see notes). |
| `force`   | `any`      |       No | Present but **unused** in current implementation.                                                      |

**Return value:** `void`.

***

## Behavior by Environment

* **Cordova/PhoneGap (iOS only):**

  * Checks `isPhoneGap() && app.device === 'iOS'`.
  * If `cordova.plugins.clipboard` exists, calls `cordova.plugins.clipboard.copy(text, onSuccess, onError)`.
  * If plugin missing → `_alert('clipboard not installed')`.

* **Web (default path):**

  * Constructs a `ClipboardItem` with:

    * `'text/html'`: `new Blob([text], { type: 'text/html' })`
    * `'text/plain'`: `new Blob([text], { type: 'text/plain' })`
  * Calls `navigator.clipboard.write([clipboardItem])`, then:

    * on **resolve** → calls `success()` if provided
    * on **reject** → `alert('error')` (does **not** call `fail`, see notes)

* **Commented legacy fallback (ClipboardJS):**

  * Code is present but commented out; would support older browsers where the Async Clipboard API is unavailable.

***

## Dependencies & Side Effects

* **Cordova path:** `cordova.plugins.clipboard` plugin (iOS); `_alert` function for plugin-missing message.
* **Web path:** `navigator.clipboard` (Async Clipboard API), `ClipboardItem`, `Blob`. Requires **secure context (HTTPS)** and typically a **user gesture**.
* **Global checks:** `isPhoneGap()`, `app.device`.
* **Side effects:** Shows a native alert `'error'` on web failure; in Cordova path, invokes provided `fail` callback on error.

***

## Usage Examples

```js
// Simple usage with callbacks
modules.clipboard.copy(
  '<b>Hello world</b>', 
  () => console.log('Copied!'), 
  (err) => console.warn('Copy failed', err)
);
```

```js
// Copy plain text only (ensure your `text` is plain)
modules.clipboard.copy(
  'Invite code: ABCD-1234',
  () => showToast('Invite code copied'),
  () => showToast('Could not copy — please press Ctrl/Cmd+C')
);
```

***

## Notes & Edge Cases (rigorous checks)

1. **Web permissions / gestures:**\
   `navigator.clipboard.write` generally requires **HTTPS** and a **user interaction** (e.g., click). Calls from timers or background scripts may be rejected.

2. **Failure callback (web path):**\
   On rejection, code currently calls `alert('error')` and **does not invoke`fail`** .\
   **Recommendation:** Replace with `if (fail) fail(error);` and avoid disruptive alerts.

3. **HTML vs. plain text:**\
   The web path writes both `text/html` and `text/plain`. Some browsers (esp. older Safari) have partial support for `ClipboardItem` and MIME types.\
   **Recommendation:** Consider feature checks and fallback to `navigator.clipboard.writeText(text)` when `ClipboardItem` isn’t supported.

4. **Android Cordova:**\
   The Cordova branch only runs for `app.device === 'iOS'`. If you need Android support, ensure the plugin is available there too and relax the device check.

5. **`force`parameter is unused:**\
   Safe to remove or implement (e.g., to force plain-text mode).

6. **Missing plugin handling:**\
   If the Cordova clipboard plugin is absent, code calls `_alert('clipboard not installed')`. Provide install guidance in dev tools/logs.

7. **Security & privacy:**\
   Browsers limit clipboard writes for security. Always prefer calling from explicit user actions (e.g., button clicks). Avoid copying sensitive tokens without consent.

8. **Legacy fallback (ClipboardJS):**\
   The commented ClipboardJS block indicates a prior fallback approach. If you must support older browsers, consider re-enabling it behind a capability check (`ClipboardJS.isSupported()`), but be mindful of maintenance and UX.

***

## Suggested Patch (optional)

If you want safer, broader support, update the web path:

```js
// Replace the web branch with capability checks and better fail handling
if ('clipboard' in navigator && 'write' in navigator.clipboard && window.ClipboardItem) {
  const items = [
    new ClipboardItem({
      'text/plain': new Blob([text], { type: 'text/plain' }),
      'text/html' : new Blob([text], { type: 'text/html'  })
    })
  ];
  navigator.clipboard.write(items).then(
    () => success && success(),
    (err) => { if (fail) fail(err); }
  );
} else if (navigator.clipboard && navigator.clipboard.writeText) {
  navigator.clipboard.writeText(text).then(
    () => success && success(),
    (err) => { if (fail) fail(err); }
  );
} else {
  // Optional: re-enable ClipboardJS fallback here
  if (fail) fail(new Error('Clipboard API not supported'));
}
```

I can provide a full drop-in replacement if you’d like.
