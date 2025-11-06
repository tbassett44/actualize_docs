---
title: Login
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
# `modules.login(options)`

Lightweight, UI-first login orchestrator that supports:

- email “next → validate” (magic-link) flow,
- password entry (toggled via flags),
- password reset & set-password,
- optional Facebook OAuth,
- and final hand-off to your app via `onLogin`.

It renders a `login` template (modal or inline), binds handlers, calls your APIs, and, on success, invokes `options.onLogin(resp)` with `{ profile, token, uid, ... }`. 

***

## Options

- `alert` (bool) – if true, shows as a modal (`$('body').alert({ template:'login', ... })`); otherwise renders into `options.ele`.
- `ele` (jQuery|Element) – container for inline render.
- `titleTemplate` (string|false) – optional title block for inline template.
- `background`, `frosted`, `backgroundColor` – visual theming for the template.
- `inline` (bool) – hint to render inline styles.
- `user` (object|false) – when present, logs in _as_ the given user id (admin/impersonation flows).
- `simplified` (bool, default `true`) – show “Next” (magic-link) first; password box appears later when needed.
- `noPlaceholder` (bool) – hide floating-input placeholders.
- `prettyClass` (string) – CSS class wrapper for input styling (default: `prettyinput`).
- `onCreate({ email })` – called if user taps “Create account”.
- `onBack()` – called when back button is tapped in inline view.
- `onLogin(resp)` – called when login **finishes** (after `/user/login` token exchange).
- `not_found_message` (string) – custom copy when email not found.
- `customProfile` (object) – forwarded to API during login.
- `force_redirect` (bool) – forwarded to OAuth flow.

***

## Public methods

- `init()`  
  Entry point: renders the UI (modal or inline), binds events (email/password, forgot, reset, FB login), resumes any in-progress magic-link validation from `localStorage.validate`, and starts polling if needed.

- `render()`  
  Inline renderer using `phi.render({ template:'login' })`; wires up:

  - email/password input “hasvalue” state,
  - “forgot password” and “back to login” toggles,
  - **set-password** UI (with optional profile-pic upload via `modules.cropuploader`),
  - buttons: **Next**, **Sign In**, **Reset**, **Create**,
  - Enter-to-submit on email/password.

- `doLogin(cb?)`  
  Thin helper that collects form data and calls `login(data, cb)`; if modal, closes on success; surfaces errors into `#signinresponse`.

- `login(data, cb)`  
  Core step. Posts to `app.sapiurl + '/user/login'` with `{ uuid, appid, source, (uid/customProfile when present) }`.  
  Behavior:

  - If server returns `success` and we have **not** shown the password yet → switch UI to password/validation phase:

    - if `app.debugallowpassword` is true → show password box immediately,
    - else → show message + start **magic-link** polling when `resp.validate` is present (`checkValidation()`).
  - If `resp.error==='user_not_found'` → set `usernotfound=1` and call `cb(false, message)`.
  - If `resp.profile` returned directly → finish via `cb(true, false, resp)`.
  - Spinner management on “Next”/“Sign In” buttons.

- `loginCallback(success, error, data)`  
  Default callback used by UI buttons:

  - on success → shows Sign In button, hides Next,
  - on `user_not_found` → reveals “no account” prompt,
  - else → prints/shakes error.

- `checkValidation()`  
  Polls `app.siteurl + '/magic_login/validate/{code}'` (≤11 minutes) passing `{ uuid }`.  
  On success → clears `validate` cache and calls `finishLogin(token, uid)`; on “already_claimed” → clears cache; otherwise re-polls every 1s.

- `finishLogin(token, uid)`  
  Exchanges magic token for a real session by posting to `app.sapiurl + '/user/login'` with `{ uuid, session: app.user.getUniqueCode(uid), token, source, appid }`.  
  On `{ profile }` → calls `options.onLogin(resp)` and any pending `cb(true, …)`.

- `reset({ un }, cb)`  
  Requests password-reset email via `app.coreapi + '/user/reset'`.  
  Responses:

  - success → `cb(true)`,
  - `user_not_found` → friendly error,
  - `must_set_password` → calls `showSetPassword(resp)` (force account claim).

- `showSetPassword(resp)`  
  Switches UI to **set-password** form; shows/hides profile-pic upload depending on `resp.hasPic`.

- `setpw({ email, p1, p2, [pic] }, cb)`  
  Finishes “must set password” flow via `app.sapiurl + '/user/setpw'`; on `{ profile }` → `cb(true, false, resp)` then `options.onLogin(resp)`.

- `bindFBLogin()`  
  Wires the “Continue with Facebook” button:

  - Launches `modules.oauth` with `{ provider:'facebook', app:'nectar', login:'fb_login' }`,
  - On success with `resp.magic` → sets `app.user.magicLink` and re-runs `app.user.get(...)` to complete onboarding/login,
  - On error → shows alert (or `_alert` in dev).

- `getClass()`  
  Returns the effective input wrapper class (options override or `prettyinput`).

- `destroy()`  
  Removes the login element from the DOM (used after success in some flows).

***

## UI states & IDs (for designers/devs)

Key elements the module reads/writes:

- `#email`, `#password`, `#resetemail`, `#setpass1`, `#setpass2`
- `#loginform`, `#forgotform`, `#setform`
- `#passwordbox`, `#signinarea`, `#nextarea`, `#noaccount`, `#getstarted`
- Response slots: `#signinresponse`, `#resetresponse`, `#setresponse`
- Buttons: `.x_next`, `.x_loginbtn`, `#reset`, `#setpw`, `.x_forgot`, `.x_tologin`, `.resetview`, `.x_create`, `.x_back`
- Optional FB: `.x_fblogin`
- Optional profile-pic upload: `.profilepicupload` (requires `modules.cropuploader`)

***

## Typical flows

**Email → Magic Link (default simplified)**

1. User enters email → clicks **Next** → `login()`
2. Server replies `{ success, validate: 'CODE' }` → UI shows pending state → `checkValidation()` polls
3. Validation success → `finishLogin(token, uid)` → `onLogin(resp)`

**Email + Password (debug/allowed)**

1. After step 2, if `app.debugallowpassword` → show password box → submit → `login()` → `resp.profile` → `onLogin(resp)`

**Forgot / Set Password**

- **Forgot** → `reset({ un })` → show success or “user not found”
- **Must set password** → `showSetPassword()` → `setpw()` → `onLogin(resp)`

**Facebook OAuth**

- `.x_fblogin` → `modules.oauth` → onSuccess with `magic` → `app.user.get(...)` → app home/welcome

***

## Integration tips / gotchas

- **Magic-link persistence:** The module caches `{ start, code }` in `localStorage.validate` and auto-resumes polling when the user re-opens the UI within 10 minutes.
- **Password path gating:** The password box is _suppressed_ unless `app.debugallowpassword` is true—intended for store-tester or internal flows.
- **Two-step `/user/login`:** It’s used both to **initiate** login (email preview) and to **finalize** session (with token/session). Keep server semantics consistent.
- **Impersonation:** Passing `options.user` sets `data.uid` on the initial call—ensure the server authorizes this path.
- **UI lifecycles:** When using modal mode (`alert:true`), the module will close the modal on success; in inline mode, it calls `options.onLogin` and leaves view management to the host.

***

## Minimal usage

```js
// Modal login
new modules.login({
  alert: true,
  onLogin(resp){
    // { profile, token, uid, ... }
    app.user.load(resp, () => app.home = new modules.home({}));
  },
  onCreate({ email }){
    // route to your signup flow
    app.router.go('/signup?email=' + encodeURIComponent(email));
  }
});

// Inline login
new modules.login({
  ele: $('#login-slot'),
  inline: true,
  background: true,
  onLogin(resp){ /* ... */ }
});
```

If you want, I can add a small **TypeScript declaration** for this API and a **contract test** that stubs the expected `/user/*` endpoints to validate success/error branches automatically.

Medical References:

1. None — DOI: file-Wnkx6N4H8zMmBDPkYrHFB3