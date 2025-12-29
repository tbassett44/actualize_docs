---
title: User
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
# `modules.user(options)`

Lightweight client-side controller for user auth/session, profile data, realtime sockets, app-level permissions, local notifications, onboarding, and a handful of UI helpers.

## Constructor options

* `store` — object to seed internal store (e.g., `{ profile, schema }`).
* `reset` — boolean; if true, clears `localStorage` token/uid on boot.
* `no_init` — boolean; skip auto-`init()` (useful for controlled bootstraps).
* `noAuth` — boolean; bypass login flow (fires `onNoAuth` / `onDoneLoading`).
* `onNoAuth()` — called when there is no valid session.
* `onDoneLoading()` — called after auth check finishes (success or not).
* `onValidAuth()` — called if session is valid (account checks are currently short-circuited).
* `onMessage(data)` — socket “relay” handler for realtime messages.
* `onLogout()` — called after a successful logout.

## Core properties

* `version` — module version number.
* `store` — persistent state container (e.g., `store.profile`, `store.schema`).
* `profile` — shorthand to `store.profile` (truthy = logged in).
* `token` — current session token (mirrors `localStorage.token`).
* `socket` — active socket.io connection (after login).
* `debugSockets` — 0/1/2; logs socket (dis)connects and traces if 2.

***

## Lifecycle & session

* `init()`\
  Starts dev socket (if `app.isdev`), loads prefs/DB, then either short-circuits (`noAuth`) or attempts `get()` and calls `validAuth()` / `onNoAuth` accordingly.

* `get(cb, force)`\
  Loads/refreshes session via `POST /user/login`, handling `temptoken`, local token, alerts/retries, and calls `load()` on success; `cb(loggedIn)`.

* `load(data, cb, loadOnly)`\
  Persists `profile`/`schema`, saves token/uid in `localStorage`, boots analytics, then `onLogin(true, cb)` unless `loadOnly`.

* `onLogin(loggedIn, cb)`\
  When logged in: set analytics user id, attach sockets (`relay`, `dev_channel`, `relay_web`), `ping()` immediately and on interval; otherwise attaches only `dev_channel` in dev.

* `logout(force, cb)`\
  Confirm (unless `force`), then `logoutAction()`.

* `logoutAction(cb)`\
  Full client reset: prefs cleared, sockets detached, amplitude reset, token wiped, `POST /user/logout`, run `onLogout`.

* `destroy()`\
  Detach socket listeners safely.

* `refresh()` / `reload(cb)` / `reRender(cb)`\
  Refresh session; or reload app UI (create `modules.home` or show `modules.welcome`).

* `validAuth()`\
  Hook for third-party account checks (currently early `return false`).

* `needsOnboarding()`\
  Returns boolean; basic gate for onboarding flows.

***

## Profile, settings & flags

* `getId()` → string\
  Returns current user id or `''`.

* `getData(key, defaultValue)`\
  Dot-path read from `store.profile`.

* `setData(data)`\
  Deep-merge `data` into `store.profile`.

* `update(save, cb)`\
  `POST /user/update` then merges `save` into profile.

* `save(collection, save, cb)`\
  `POST /user/save` (generic user collection write).

* `getSetting(setting)`\
  Reads nested `profile.settings` (supports dot paths).

* `setSetting(setting, value)`\
  Optimistically updates local `profile.settings`; server `set()` with `{ app:'user_settings' }`.

* `setFlag(flag, value)`\
  Set `profile.flags[flag] = value`; server `set()` with `{ app:'user_flag' }`.

* `set(data, cb)`\
  Generic `POST /user/set` (requires `token`).

* `profileUpdate()`\
  Refreshes main username/pic and level labels in the DOM.

***

## Scopes & roles

* `getScopes()`\
  Aggregate of `profile.scopes_info[].name` plus the user’s own id; `'*'` means super-scope.

* `hasScope(check_scopes, mergeData)`\
  Returns `1/0`. Accepts a string or array. Supports dynamic scope substitution like `["[some.dot.path]"]` against `mergeData`.

* `getRoles()`\
  Returns `profile.roles` array or `[]`.

***

## Identity menu helpers

* `getIdentity()`\
  Resolves current posting identity: user profile or a selected page; falls back to profile.

* `getIdentities()`\
  Returns an array of `{ id,name,pic }` for profile + pages list.

***

## Scheduling & local notifications

* `ensureSchedule()`\
  Computes upcoming triggers for all saved schedules and registers missing local notifications.

* `getUpcomingSchedule(schedule)`\
  Expands `times` × `frequency` over `maxScheduleLength` days into future trigger timestamps.

* `getOptionHash(options)`\
  Base64 hash used to detect schedule option changes.

* `hasScheduleItem(scheduled, key, time, options)`\
  Checks if a trigger exists and matches options (otherwise unschedules).

* `scheduleItem(options)`\
  Registers a local notification via `phone.localnotification.register`, persists to prefs.

* `unScheduleItem(id)`\
  Removes a scheduled local notification.

* `clearSchedule(type)` / `hasSchedule(type)` / `getSchedule(type)`\
  CRUD helpers for schedule definitions in prefs.

* `schedule(type, options)`\
  Create/update or remove a schedule definition and re-`ensureSchedule()`.

***

## Realtime, dev hot-reload & UI

* `startSocket(id, func)` / `stopSocket(id, func)`\
  Attach/detach named socket listeners with de-dupe logging.

* `onMessage(data)`\
  Forwards socket payloads to `options.onMessage`.

* `onDevMessage(data)`\
  Dev-only handler (e.g., hot-reload signals).

* `reloadComponent(pathParts)` / `hotReload(component)`\
  Dev file watcher: reloads JS/CSS/templates or re-fetches module dependency, then calls `modules.viewdelegate.onHotReload()` and `phi.onHotReload()`.

* `toggleText()`\
  Toggles `body.hidehelptext`.

* `reloadModal()` / `updateModal()`\
  System modals to reload app (auto or user-triggered).

* `setPage(path)`\
  Track pageview (Google Analytics).

***

## Network pings & updates

* `ping()`\
  `POST /user/ping` with app/bootloader metadata; if a newer core is available, passively reload core and open `updateModal()`. Also updates splash assets and time sync.

* `pingInterval()`\
  Returns ms (10s dev / 30s prod).

***

## Account linking & secure web handoffs

* `checkIfAccountExists(type, cb)`\
  `POST /user/hasaccount` to verify linked account.

* `ensureAccountCreation({type,url})`\
  Modal prompting user to create/check a third-party account; opens `url` and rechecks on close.

* `secureLinkToWeb(url, cb)`\
  Fetches account token (`/user/accounttoken`) and opens a secure link (adds `token` and optional `scheme`).

* `forceUserAccountUpdate(cb)`\
  Modal requiring account update or logout; uses `secureLinkToWeb()` and schedules a post-update re-login.

***

## Auth utilities

* `promptLogin()`\
  Simple “become a member to comment!” alert.

* `addUserQS(clean)`\
  Returns URL query string with `token`, `session`, `appid` if logged in.

* `getUniqueCode(uid?)`\
  Generates an obfuscated session code from `uid` and device UUID.

***

## Permissions API (`user.permissions`)

* `permissions.current` — live status cache `{ geolocation?, push? }`.

* `permissions.getAvailable()`\
  Returns the set of permission keys relevant to the platform.

* `permissions.checkAll(cb)`\
  Parallel check of all platform permissions (cordova-diagnostic).

* `permissions.hasPermission(type, cb)`\
  Resolves to current permission state; always `true` on web.

* `permissions.ask.geolocationAccuracy(cb)`\
  Returns `FULL` / `REDUCED` or `false`.

* `permissions.ask.geolocation(cb, fine?)`\
  Requests location auth; if granted, follows up with accuracy authorization.

* `permissions.ask.push(cb)`\
  Requests remote notification auth (iOS).

* `permissions.canCheck()`\
  True if PhoneGap + diagnostic plugin available.

* `permissions.hasAll(onComplete)`\
  Quick “all granted?” boolean check using the cached `current` map.

* `permissions.ensure(singleSuccess, onComplete)`\
  Prompts for any missing permissions (sequential), calling `singleSuccess(key)` on each newly granted one.

* `permissions.start(ele)`\
  Convenience: ensure and update a UI element’s per-permission state.

* `permissions.onboard(ele, ocb)`\
  Renders or navigates to a “permissions page” and walks the user through granting; calls `ocb` when done.

***

## Miscellaneous

* `saveStat(opts)`\
  Sends a usage stat (`/user/savestat`), auto-filling `page` with either user id or `anon`.

* `getBanner()`\
  Returns a CTA banner config (e.g., “Become a Player!”) or `false`.

* `onResume()`\
  On app resume, calls `ping()`.

***

## Quick usage

```js
// initialize (default flow)
app.user = new modules.user({
  onNoAuth(){ /* show login */ },
  onValidAuth(){ /* session is valid */ },
  onDoneLoading(){ /* hide splash */ },
  onMessage(evt){ /* socket message */ }
});

// ensure the user is loaded (if you disabled auto init)
const u = new modules.user({ no_init: true });
u.init();

// read profile safely
const name = u.getData('profile.name', '(guest)');

// update a user setting
u.setSetting('notifications.push.enabled', true);

// request permissions during onboarding
u.permissions.onboard(null, () => console.log('perms complete'));
```

***

### Notes / gotchas to consider

* **Token storage**: Token and uid are stored in `localStorage` (`token` mirrors `_id`), so treat the API as session-token based.
* **UI coupling**: Several methods manipulate DOM directly (`profileUpdate()`, modals). If you use this module outside the main app shell, stub or wrap these calls.
* **Dev reloads**: Hot-reload paths assume your asset server and `phi.ensureDependencies` plumbing are present in dev.
* **Error surfacing**: Login failures can present alerts; if you run headless, override `onNoAuth`/`onDoneLoading`.
