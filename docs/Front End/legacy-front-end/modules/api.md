---
title: API
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
# Module: `modules.api` (AJAX/request orchestration helper)

## High-Level Summary

`modules.api(topts)` is a thin wrapper around **jQuery’s`$.ajax`** plus a few convenience behaviors: it enriches outgoing data with app/session tokens, supports optional **window navigation (`open`)**, environment forcing via `localStorage`, retry-on-failure semantics, duplicate-request guarding via a `key`, developer JSONP mode, button “sending” states, and timing/diagnostics. It unifies success/error callback invocation and centralizes where your UI can attach loading modals or pre-call hooks.

***

## Options / Props

> Unless noted, all options are optional. Types inferred from usage.

| Name            | Type                              | Required | Default                     | Accepted Values / Shape | Description                                                                                                         |   |                                                                         |
| --------------- | --------------------------------- | -------: | --------------------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------- | - | ----------------------------------------------------------------------- |
| `url`           | `string`                          |  **Yes** | —                           | —                       | Endpoint URL. If it starts with `app.apiurl2`, the method is forced to `GET`.                                       |   |                                                                         |
| `data`          | `object`                          |       No | `{}`                        | —                       | Payload. Auto-enriched unless `cleanData` is true.                                                                  |   |                                                                         |
| `open`          | `boolean`                         |       No | `false`                     | —                       | If true, **navigates** instead of AJAX. Top-level branch uses `'_self'`; merged `opts.open` branch uses `'_blank'`. |   |                                                                         |
| `loadingmodal`  | `object`                          |       No | —                           | `{ uid?: string, ... }` | If present and missing `uid`, a `uid` is assigned (`Math.uuid(8)`); not otherwise used here.                        |   |                                                                         |
| `cleanData`     | `boolean`                         |       No | `false`                     | —                       | Skip auto-injection of `appid`, `token`, and `session`.                                                             |   |                                                                         |
| `type`          | `"GET" \\| "POST" \\| string`     |       No | `'POST'`                    | —                       | HTTP method. Overridden to `'GET'` when `url` starts with `app.apiurl2`.                                            |   |                                                                         |
| `dataType`      | `'json' \\| 'jsonp' \\| string`   |       No | `'json'` or `'jsonp'` (dev) | —                       | In dev (\`app.debugapi                                                                                              |   | app.isdev`) and when `topts.dataType != 'json'`, coerced to `'jsonp'\`. |
| `timeout`       | `number`                          |       No | `8000`                      | ms                      | XHR timeout.                                                                                                        |   |                                                                         |
| `jsonpCallback` | `string`                          |       No | `'jcb_'+Math.uuid(4)`       | —                       | JSONP callback name.                                                                                                |   |                                                                         |
| `btn`           | `jQuery`                          |       No | —                           | —                       | If provided, gets `.addClass('sending')` before request and removed on success.                                     |   |                                                                         |
| `key`           | `string`                          |       No | `''`                        | —                       | Deduplication key stored in `app.api_sending[key]` (see notes).                                                     |   |                                                                         |
| `retry`         | `number`                          |       No | —                           | > = 0                   | Automatic retries on error (`timeout` branch uses different path). Decremented per attempt.                         |   |                                                                         |
| `forceRetry`    | `boolean`                         |       No | `false`                     | —                       | Internal flag: after an `xhr.readyState===0` error, one immediate re-attempt is made.                               |   |                                                                         |
| `precall`       | `function()`                      |       No | `() => {}`                  | —                       | Hook executed just before initiating the request.                                                                   |   |                                                                         |
| `callback`      | `function(response, topts, xhr?)` |       No | —                           | —                       | Called on success; also called on handled errors (see Error Handling).                                              |   |                                                                         |
| `success`       | `function(data, textStatus, xhr)` |       No | internal                    | —                       | Internal success handler; ultimately invokes `callback`.                                                            |   |                                                                         |
| `error`         | `function(xhr, status, reason)`   |       No | internal                    | —                       | Internal error handler; may retry and/or invoke `callback`.                                                         |   |                                                                         |
| `onecallback`   | `boolean`                         |       No | `false`                     | —                       | Present but not used in this implementation.                                                                        |   |                                                                         |
| `ele`           | `any`                             |       No | `false`                     | —                       | Present but not used in this implementation.                                                                        |   |                                                                         |
| `startTime`     | `number`                          |       No | set internally              | epoch ms                | Request start timestamp (used for diagnostics in errors).                                                           |   |                                                                         |

### Auto-enriched fields (when `cleanData` is **false**)

* `appid`: from `app.appid` (if missing on `data`)
* `token`: from `app.user.token` → `app.token` → `localStorage('temporary_token')` (created via `Math.uuid(16)` if missing)
* `session`: from `app.user.getUniqueCode()` if available
* `forceenv`: from `localStorage('env')` (only when **not** `app.isdev`)

***

## Events / Callbacks

| Hook                 | Params                    | When It Fires                   | Notes                                                                                                                                                                                                                                  |
| -------------------- | ------------------------- | ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `precall`            | `()`                      | Right before request dispatch   | Good for logging / last-minute data shaping.                                                                                                                                                                                           |
| `success` (internal) | `(data, textStatus, xhr)` | On XHR success                  | Clears `app.api_sending[key]`, removes `btn.sending`, then calls `callback(data, topts)`.                                                                                                                                              |
| `error` (internal)   | `(xhr, status, reason)`   | On XHR failure                  | Manages retries. On `timeout`, calls `callback({error:'Network Timeout', type:'internet', stack?})` (stack only in dev). On `readyState===0`, may force a one-time immediate retry; otherwise calls `callback` with `{error, status}`. |
| `callback` (user)    | `(payload, topts, xhr?)`  | On success **or** handled error | Your central result handler.                                                                                                                                                                                                           |

***

## Dependencies & Side Effects

* **External:** jQuery (`$.ajax`, `$.param`), global `app` (fields: `appid`, `user`, `token`, `apiurl2`, `api_sending`, `isdev`, `debugapi`, `netfail`), `Math.uuid`, `localStorage.getVar/setVar`, global `onerror` logger (called in some error paths).
* **UI side effects:** Adds/removes `.sending` on `btn`. May log warnings / emojis to console. May navigate via `window.open`.
* **Global state touched:** `app.api_sending[key]` (for de-dup), `app.netfail` (timeout id), `localStorage('temporary_token')`, `localStorage('env')`.

***

## Option Details

### `open`

* **Top-level branch:** If `topts.open` is truthy **before** options are merged, the function **navigates to** `topts.url + '?' + $.param(topts.data)` in the **same tab** (`'_self'`) and returns.
* **Merged`opts.open` branch:** After defaults+merge, if `opts.open` is truthy, it opens the same URL in a **new tab** (`'_blank'`) and returns.
* **Consideration:** Passing sensitive fields (like `token`) via query string can leak credentials in browser history, logs, referers.

### Dev JSONP mode

* When `app.debugapi || app.isdev` **and** `topts.dataType != 'json'`, `dataType` is coerced to `'jsonp'`. That implies **GET** semantics and a `jsonpCallback`. Ensure your API supports JSONP in those contexts.

### Method override by `apiurl2`

* If `topts.url` starts with `app.apiurl2`, the request method is **forced to`'GET'`** . This can silently discard POST bodies; make sure this is intended.

### Retries

* If `topts.retry > 0`, on error it decrements and schedules a **5s retry** (`setTimeout(modules.api, 5000)`).
* For `xhr.readyState===0`, the code performs a **single immediate retry** by setting `forceRetry=1` and calling `modules.api(topts)` (no delay).
* Calls to `callback` are gated; if there’s no `callback`, non-timeout errors may be swallowed (see notes).

### Duplicate request guard (`key`)

* If `topts.key` is present, it sets `app.api_sending[key]=1` on dispatch, and deletes it on success/error. If the key already exists, it only logs a warning but **does not bail out**—the request still fires (see notes).

***

## Usage Example

```js
// Basic POST with auto-injected app/session tokens
modules.api({
  url: app.apiurl + '/profile/update',
  data: { name: 'Juicy' },
  retry: 2,
  btn: $('#saveProfile'),
  precall() {
    console.log('Dispatching profile update…');
  },
  callback(res) {
    if (res && res.error) {
      return alert('Error: ' + res.error);
    }
    alert('Saved!');
  }
});
```

```js
// GET with deduplication key and explicit JSON dataType in dev (avoid JSONP)
modules.api({
  url: app.apiurl + '/events/list',
  type: 'GET',
  dataType: 'json',          // prevents dev override to 'jsonp'
  key: 'events:list',        // guard against parallel duplicates
  retry: 1,
  cleanData: false,          // include tokens/session
  callback(res) {
    // handle list
  }
});
```

```js
// Navigate instead of AJAX (opens in new tab)
modules.api({
  url: app.siteurl + '/export',
  data: { format: 'csv' },
  open: true                 // WARNING: query contains data; avoid tokens
});
```

***

## Notes & Edge Cases (and where to poke holes)

1. **Token leakage via query strings**\
   Both `open` paths append `$.param(data)` to the URL. If `cleanData` is false (default), this may include `token` and `session`. That’s risky (referer leaks, logs, browser history).\
   **Recommendation:** When `open` is true, automatically **omit** sensitive fields or force `cleanData: true`, and whitelist only safe params.

2. **Inconsistent`open` targets**\
   Top-level `topts.open` opens `'_self'`, while merged `opts.open` opens `'_blank'`. This is surprising.\
   **Recommendation:** Standardize to one behavior (configurable `openTarget?: '_self'|'_blank'`).

3. **Forced`'GET'` for`apiurl2`**\
   Silently coercing method can break endpoints expecting `POST` bodies.\
   **Recommendation:** Make this opt-in or add an assertion/log warning when coercion happens.

4. **Dev JSONP coercion**\
   In dev, unless `topts.dataType === 'json'`, you’ll get JSONP (GET). This can unintentionally alter server semantics or break CORS expectations.\
   **Recommendation:** Default to `'json'` unless explicitly requested; or gate JSONP behind `enableJsonp: true`.

5. **Duplicate request guard does not stop duplicates**\
   If `app.api_sending[key]` exists, the function **logs** but still proceeds.\
   **Recommendation:** `return false;` (or queue) when a duplicate is detected.

6. **Callback silence on some errors**\
   If there’s **no`callback`** and the error isn’t a timeout, the handler falls through doing almost nothing (just comments).\
   **Recommendation:** Always surface an error via a default handler or promise-style API.

7. **Global timeout handle (`app.netfail`)**\
   Retries use a single global timeout var; concurrent requests may **clobber** each other’s timers.\
   **Recommendation:** Store timeout per `key` or per `startTime` to avoid overwriting.

8. **Immediate retry on`readyState===0`**\
   This may create two overlapping flows when combined with `retry` logic (one immediate, then scheduled).\
   **Recommendation:** Consolidate retry strategy (uniform backoff with cap).

9. **Security: token in localStorage + temporary token**\
   Temporary token is generated and persisted when no token is present. Ensure server treats this as non-privileged and that endpoints won’t leak data.\
   **Recommendation:** Document server expectations; consider scoping/expiring temporary tokens.

10. **Unused options**\
    `onecallback` and `ele` are defined but unused; consider removing or implementing.

11. **Error message typos / logging**\
    `"Unkonwn API Error"` typo; `onerror(...)` is referenced but not guaranteed to exist.\
    **Recommendation:** Correct typos and guard `onerror` or replace with a safe logger.

12. **Button state not cleared on error**\
    `btn.removeClass('sending')` is only in `success`. On error (without retry), button may remain in sending state.\
    **Recommendation:** Clear on final error path as well.

13. **No abort mechanism**\
    There’s no returned jqXHR or abort handle.\
    **Recommendation:** Return the jqXHR from `$.ajax(opts)` to allow cancellation.

***

## Dependencies & Side Effects (Quick Reference)

* **Relies on:** `jQuery`, `app` (numerous fields), `Math.uuid`, `localStorage.getVar/setVar`
* **Mutates:** `app.api_sending`, `app.netfail`, `localStorage('temporary_token')`
* **UI:** `.sending` class on `btn`
* **Navigation:** may call `window.open(...)`

***

## Suggested Safer Wrapper (optional improvement)

If you want, I can refactor this into a **Promise-based** helper that:

* Returns an object `{ jqXHR, cancel(), finished }`
* Unifies retry/backoff
* Sanitizes `open` parameters automatically
* Deduplicates by `key` (no-op if in-flight)
* Ensures button state resets in **all** terminal paths

Say the word and I’ll provide a drop-in patch.
