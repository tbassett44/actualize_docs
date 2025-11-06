---
title: Common Tools
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
# Frontend Tools (tools.js) — Technical Reference

## Overview

This module exposes a global toolbox on `window`:

- **`df(obj, path, def)`**: safe deep getter (supports one set of `[...]` array brackets).
- **`ds(obj, path, val)`**: safe deep setter (creates parents along the path).
- **`modules.tools` (aliased to `window._`)**: a large utility suite for DOM, URLs, timers, env/device, content formatting, images, geolocation, money math, link routing, file loading, iframes, etc.

## External Dependencies & Globals

Used throughout—make sure these are available where relevant:

- **jQuery** (`$`)
- **async** (for parallel loads)
- **anchorme** (linkifying plain text)
- **JSONFormatter**
- **GraphemeSplitter**
- **tinycolor** (commented/mentioned)
- **Math.uuid** (ID generation)
- **localStorage extensions**: `getObject`, `setObject`, `getVar`
- **Global `app` object**: `{ s3, imgurl, siteurl, sapiurl, apiurl, envs, endpoint, appid, user.token, device, themeColor, time, history, ... }`
- **`modules.api`, `modules.toast`, `modules.clipboard`, `modules.geocode`**
- **Cordova** (optional paths): `cordova.InAppBrowser`, `cordova.plugins.permissions`, `window.plugins.socialsharing`, `window.plugins.CallNumber`, `SafariViewController`, `isPhoneGap()`, `phone.statusBar`
- **Global flags**: `isMobile`.

***

## API Reference (most-used / most-critical)

### Deep Accessors

- **`df(obj, path, def)`** → any  
  Safe deep read. Supports a single bracket section like `"a[2].b"` as well as dot paths `"a.b.c"`. Returns `def` if missing.

  ```js
  const count = df(summaryData,'visitors.graphs.active.count', 0);
  const item  = df(obj, 'list[1].name', 'Unknown');
  ```
- **`ds(obj, path, val)`** → `obj`  
  Safe deep write. Creates intermediate objects if needed.

  ```js
  ds(profile, 'settings.theme.color', '#000');
  ```

### Object/Path helpers

- **`_.dotGet(path, obj)`** → any
- **`_.dotSet(path, value, obj)`** → `obj` (thin wrapper over `ds`)
- **`_.parseString(str, data)`** → string  
  Replaces `[a.b.c]` placeholders using `dotGet` in `data`.

### Collections & Sorting

- **`_.intersect(arr1, arr2)`** → array (intersection)
- **`_.size(obj)`** → number (keys/length or `0`)
- **`_.ksort(obj)`** → new object with sorted keys
- **`_.arsort(arr, key)`** → array of ids by descending `sort` on objects
- **`_.kGetSort(obj, count)`** → sorted keys

### Timers

- **`_.setInterval(id, fn, ms)`** / **`_.clearInterval(id)`** / **`_.clearIntervals()`**  
  Named interval management.
- **`_.throttle(id, ms, cb, noclear)`**  
  Simple timeout-based throttle keyed by `id`. Pass `false` as `ms` to cancel.

### Logging & IDs

- **`_.log(msg, type?)`**  
  Styled console logging; types: `navigation`, `socket`, `time`, `cookie`, `stats`, `file`, `keyboard`, default.
- **`_.getUniqueId()`**  
  Uses `Math.uuid(6)` and avoids collisions with an internal `ids` array.

### Layout & UI

- **`_.isWebLayout()`** → boolean  
  `true` if not PhoneGap and `body.width >= 802`.
- **`_.fitContent(el)`**  
  Autoresizes font within width/height constraints via `data-*` attributes (e.g., `data-max`, `data-min`, `data-width`, `data-height`, `data-center`, `data-incriment`).
- **`_.wrapContent(html, length, startExpanded?)`**  
  Collapsible “Read More / Show Less” wrapper that tries not to break inside nested tags.
- **`_.trimContent(text, length)`**  
  Strips HTML tags and trims to length (aware of `</p>`).

### Content Formatting / Links

- **`_.fixContent(text, opts)`** → HTML string  
  Rich formatter that:

  - Linkifies text via **anchorme** (or works with rich HTML input).
  - Converts links to `<span class="linknav" data-type="external" data-intent="...">` for internal routing.
  - Highlights phone numbers with `.x_phone`.
  - Enforces `https:` on `//` iframes; strips inline `width`/`height` on `img`.
  - Options: `stripHtml`, `redactor`, `fontColor`, `fontSize`, `classes`, `maxlength`, `truncate`.

- **`_.openLink(e, events?, skip?)`**  
  Centralized link router handling:

  - `tel:` links (PhoneGap-aware)
  - External links (new tab / InAppBrowser/SafariViewController on mobile)
  - Internal routing via `app.history`
  - “inappbrowser” type with optional `events` hooks (`opened`, `load`, `closed`)
  - Mailto → native email on Cordova, or `mailto:` fallback

- **`_.wrapExternalLink(url)`** / **`_.unwrapExternalLink(url)`**  
  Wrap/unwrap external URLs via `app.sapiurl/externallink?u=...`.

- **`_.getqs(url)`** and **`_.getQsVars(url)`** / **`_.getQsVar(url, key)`**  
  Query-string parsing helpers.

- **`_.highlightString(str, search, class?)`**  
  Simple case-insensitive highlight.

### Images & Files

- **`_.getUserImg(uid, type)`** → URL
- **`_.getImg(objOrUrl, type, forceGroot?)`** → URL

  - Supports objects with `{ url }` or S3-style `{ path, ext, v }`.
  - Falls back to proxy if plain `http` (non-https) URL.
- **`_.getImgHeight(meta, sizes, asStyle?)`**  
  Calculates consistent display dimensions from aspect ratio and constraints.
- **`_.getFile(obj, type='media_full')`** → URL
- **`_.downloadFile(url)`** → triggers hidden iframe download.

### Environment & Device

- **`_.getTimeZone()`** → IANA zone (uses `Intl` if available; defaults to `America/Denver`)
- **`_.setFaviconCount(count, flash?)`**  
  Badge on favicon (auto flash/reset).
- **`_.getEndpoint(env?)`** → URL  
  Resolve API endpoint by environment name from `app.envs` or `app.endpoint`.
- **`_.ensureAndroidPerms(types[], cb)`**  
  Cordova Android permissions request loop.
- **`_.audioContext.get/set/release()`**  
  iOS Safari audio context management.

### Location (best-available)

- **`_.location.start()`** → kicks `loadIpLocation()`
- **`_.location.init()`** → (PhoneGap only) tries GPS and sets `.data.geo`
- **`_.location.get(cb, fcb)`** → memoized locate call
- **`_.location.getNearestLocation()`** → `geo` → `ip` → `false`
- **`_.location.getName(data, type)`** → string label via `modules.geocode.getText(data)`
- **`_.location.loadIpLocation()`** → calls `modules.api` `/iplocation`
- **`_.location.locate(cb)`** → `navigator.geolocation.getCurrentPosition`

### Math / Compare / Geo

- **`_.deepCompare(a, b, ...)`** → boolean  
  Deep equality with cycle detection; handles `Date`, `RegExp`, functions.
- **`_.toRad(deg)`**, **`_.calcCrow(lat1, lon1, lat2, lon2, km)`**  
  Haversine distance (returns km by default).

### Sharing / Email

- **`_.share(opts)`**  
  On web: copies `opts.url` to clipboard and toasts; on Cordova: uses `socialsharing`.
- **`_.sendEmail({ to, subject, content })`**  
  Cordova email or `mailto:` fallback.

### Iframe Messaging

- **`_.iframe.sendChild(iframe, data, origin='*')`**
- **`_.iframe.sendParent(data)`**
- **`_.iframe.listenChild(cb, id?)` / `.unListenChild(id)`**
- **`_.iframe.listenParent(cb, id?)` / `.unListenParent(id)`**

### Money / Fees

- **`_.fromMoney('$str')`** → cents (int)
- **`_.toMoney(cents, commas?, add?, stripZeros?)`** → string
- **`_.calcPlatformFee(amountCents, formulaString?, noFee?)`**
- **`_.calcStripeFee(amountCents)`**, **`_.calcStripeTapFee(amountCents)`**  
  (See notes on accuracy under Issues.)
- **`_.getMoneyColor(val, tx?, me?, isPositive?)`**, **`_.getMoneySign(val, tx?, me?, isPositive?)`**

### Loading assets

- **`_.addFiles.load({ js:[], css:[] }, cb)`**  
  Injects CSS (as `<style>`) and JS (via `<script>`), with timeouts and deduping.
- **`_.addFiles.css(url, time, scb, fcb)`** / **`_.addFiles.js(...)`**

***

## Known Issues & Fixes (recommendations)

1. **`_.getObjectDiff(obj1, obj2)` — “additions” loop bug**  
   The second loop iterates `obj1` again instead of `obj2`, so additions in `obj2` are never reported.  
   **Fix:** iterate `obj2` in the second pass:

   ```js
   for (const key in obj2) {
     if (!obj1.hasOwnProperty(key)) diff[key] = { type: 'add', to: obj2[key] };
   }
   ```

2. **`_.getInjectCode()` uses `self` that isn’t defined**  
   It references `self[functionname]`; within this scope `self` isn’t declared.  
   **Fix:** change to `modules.tools[functionname]` (or bind `const self = modules.tools;`).

3. **`_.calcStripeFee` unreachable code**  
   Two `return` lines—second is never reached. Also, fee math differs by region & pricing tier.  
   **Fix:** keep one formula and add unit tests; consider param for fee schedule.

4. **`_.getImg` type check typo and fallbacks**  
   `typeof obj=='obj'` is never true; should be `typeof obj==='object'`. Also, ensure `app.s3` exists before concatenation; else return original URL.

5. **`_.getImgHeight` variable scoping**  
   Some branches use `h` without declaration or return it inconsistently.  
   **Fix:** ensure `let h` is declared where needed and consistently returned.

6. **`_.getPlanInfo(user, status)` references `data`, `info` that don’t exist**  
   This function is incomplete and returns early; downstream code will break if called.  
   **Fix:** either finish the implementation or remove/export a stub that throws with guidance.

7. **`_.openLink` security considerations**

   - For external links, you’re wrapping/unwraping URLs—good.
   - **Add:** `rel="noopener noreferrer"` when using `window.open(...,'_blank')` on web to avoid tabnabbing.
   - **Validate:** Only allowlist external protocols (`http:`, `https:`, `mailto:`, `tel:`) before opening.

8. **XSS surface in `_.fixContent` / `_.wrapContent`**  
   You sanitize some HTML, but you also inject content back via `.html()`.  
   **Fix:** If input comes from users, sanitize with a robust library (e.g., DOMPurify) before calling these.

9. **Cordova feature detection**  
   Many paths are checked, but add more defensive checks (e.g., `if (window.SafariViewController && SafariViewController.show)`).

10. **`df` bracket handling**  
    Only supports one `[]` segment per key portion. The doc warns about it—good—but consider extending or clarifying in docs where that’s a hard constraint.

11. **Performance notes**

    - `_.fixContent` walks DOM and regexes; cache where possible for repeated render paths.
    - `_.wrapContent` computes indices and rebuilds DOM—avoid on very long strings when unnecessary.

***

## Usage Patterns & Examples

### Internal navigation with `linknav` spans

```html
<span class="linknav" data-type="external" data-intent="https://example.com">Open</span>
<script>
  $('.linknav').on('click', function () { _.openLink($(this)); });
</script>
```

### Safe deep reads/writes

```js
const val = df(app, 'user.profile.handle', 'anonymous');
ds(app, 'user.profile.flags.isBeta', true);
```

### Formatting user content for display

```js
const html = _.fixContent(userInput, { classes: 'prose linknav', maxlength: 42, truncate: 240 });
container.innerHTML = html;
// Then delegate clicks for .linknav to _.openLink
```

### Device-aware flow

```js
if (_.isWebLayout()) {
  // desktop layout
} else {
  // mobile / PhoneGap layout
}
```

### Throttled UI work

```js
window.addEventListener('resize', () => {
  _.throttle('layout', 200, () => recomputeLayout());
});
```

### Iframe postMessage

```js
_.iframe.listenParent((e) => { /* handle */ }, 'my-channel');
_.iframe.sendChild(document.getElementById('childFrame'), { type: 'PING' }, '*');
```

***

## Test Checklist (high-value)

- **Routing:** `_.openLink` for `tel:`, `mailto:`, internal (`app.history`), and external (`_blank`, InAppBrowser).
- **Sanitization:** Feed `_.fixContent` risky inputs; verify no executable HTML survives.
- **Image URLs:** `{ url }`, S3 object, raw http, https, and `app.nointernet` flag.
- **DF/DS edge cases:** Missing parents, array bracket reads, default values.
- **Location:** Geolocation timeout path, `loadIpLocation` failure handling.
- **Cordova:** Permissions flow on Android; social sharing/email fallbacks.
- **Money math:** Known amounts (e.g., 100, 500, 10_000 cents) vs expected stripe/platform fees.

***

## Quick Win Fixes (safe to ship now)

1. Patch **`getObjectDiff`**, **`getInjectCode`**, **`calcStripeFee`**, **`getImg`** type check.
2. Add protocol allowlist + `noopener` on external `window.open`.
3. Wrap `_.fixContent` outputs with a sanitizer if any user-generated input is possible.

If you want, I can push a PR-style diff with these fixes and add JSDoc blocks right into the file (df, ds, openLink, fixContent, getImg, getObjectDiff, getInjectCode), plus a tiny Jest suite for the hotspots.