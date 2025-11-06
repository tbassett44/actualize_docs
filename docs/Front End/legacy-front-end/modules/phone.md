---
title: Phone
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
# Module: `window.phone` — Cross-device device & OS integrations

> A unified façade over calling, calendars, location, badges/notifications, camera & uploads, orientation, status bar, deep-links, push, and more—designed to run in **Cordova/PhoneGap** environments and degrade gracefully on the web. 

***

## Top-level

| API                              | Purpose                                                                                                                                                                                                                    |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `phone.version`                  | Internal version flag.                                                                                                                                                                                                     |
| `phone.log(msg)`                 | Tag logs with `[PHONE]`.                                                                                                                                                                                                   |
| `phone.init()`                   | App bootstrapping: starts location, amplitude analytics, calendar sync, sets platform/body classes, binds app lifecycle (`resume/pause/backbutton`), prepares keyboard/status bar, Mobiscroll theme, etc. (Cordova-aware). |
| `phone.parseURL(url)`            | Handle app-scheme deep links; routes via `app.history`.                                                                                                                                                                    |
| `phone.track(name, props)`       | Amplitude wrapper.                                                                                                                                                                                                         |
| `phone.background.{clear,reset}` | Temporarily hide UI behind a transparent background (e.g., screenshots/recording interop).                                                                                                                                 |

***

## Calls

| API                  | Behavior                                                                                                                                             |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `phone.call(number)` | If Cordova: uses `window.plugins.CallNumber` to dial. Accepts a string **or** `{code, number}` object (concatenated). Logs when not available (web). |

***

## Calendar (device calendar integration)

| API                                                          | Behavior                                                                                                                                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `calendar.linkCalendar(cb, opts, force)`                     | Prompts user to pick a default calendar (unless already set), stores ID in prefs, returns calendar via `cb`.                                                       |
| `calendar.getCalendar(cb, topts, force, silent)`             | Core picker; lists device calendars via `window.plugins.calendar.listCalendars`, respects a stored default, and can show a picker (`mobilealert_calendar_picker`). |
| `calendar.ensureLocalCalendar(immediate)`                    | Background sync: fetches server feed (`/module/calendar/feed`) and mirrors RSVP’d events into the user’s device calendar; updates/creates/deletes as needed.       |
| `calendar.addEvent(opts, update?, silent?, cb?)`             | Create event with `createEventWithOptions` (handles calendarName/Id per platform, reminders, custom `calOptions`).                                                 |
| `calendar.editEvent(opts, silent?, cb?)`                     | Replace (delete-then-add) an existing event by stored id.                                                                                                          |
| `calendar.deleteEvent(opts, cb?)`                            | Delete by stored native id (`deleteEventById`) using a cached mapping in prefs (`event.{id}`).                                                                     |
| `calendar.addSingleEvent(event)`                             | Convenience: derive `opts` from an event object, ensure default calendar, and upsert.                                                                              |
| `calendar.getDefaultCalendar(cb)` / `clearDefaultCalendar()` | Manage default calendar selection.                                                                                                                                 |
| `calendar.getCalendarData(data)`                             | Categorize calendars into **Local / Recent / Online (CalDAV)** groups with display orders.                                                                         |
| `calendar.setRecent(calendar)`                               | Maintain MRU list of calendar ids.                                                                                                                                 |
| `calendar.setLast(ts)` / `getLast()`                         | Track last sync timestamp in prefs.                                                                                                                                |

**Notes**

* iOS/Android differences handled via `calendarName` vs `calendarId`.
* Sync only for relevant events (e.g., RSVP going/interested/host) and sets 15-minute reminder by default.
* Uses `modules.api` for server feed; shows toasts on batch update.

***

## Location (GPS/IP/city selection + helpers)

| API                                | Behavior                                                                                                                                        |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `location.start()`                 | Bootstraps IP-based location via `/iplocation`.                                                                                                 |
| `location.init(retry?)`            | Cordova-only GPS init: binds selection guard (text selection disables swipe gestures) and fetches geolocation with timeout; fires `onGpsStart`. |
| `location.get(cb, fcb)`            | Single-shot GPS fetch with in-flight guard.                                                                                                     |
| `location.getNearestLocation()`    | Priority: user-picked **city** → **GPS** → **IP** → `false`.                                                                                    |
| `location.setCity(featureOrFalse)` | Lock proximity bias to a selected Mapbox place (stores center as `{lng,lat}`).                                                                  |
| `location.getName(data, type)`     | Format helper using `modules.geocode.getText` (e.g., `"city"`, `"city_full"`, `"simple"`).                                                      |
| `location.locate(cb)`              | Raw `navigator.geolocation.getCurrentPosition` with \~5s timeout and accuracy threshold.                                                        |

**Tip:** This integrates with your Mapbox geocoder wrapper for uniform place labeling. 

***

## Badge (unread counts & app icon badge)

| API                                                                    | Behavior                                                                                     |
| ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `badge.init({notificationEle, notificationBadge, chatEle, chatBadge})` | Fetch counts from `/user/badge`, subscribe to socket `"badge"` events, and paint DOM badges. |
| `badge.set([data])`                                                    | Update DOM + native badge (Cordova). Caps counts at 99.                                      |
| `badge.removeChat(chatId)`                                             | Clear a specific chat from local badge model and repaint.                                    |
| `badge.getTotal(skip_identity?)`, `badge.getChatCount(chatId)`         | Counters over the internal badge model.                                                      |

**Identity keying**: uses `getIdentity(id)` to namespace per-user/group. 

***

## Camera & Uploads

| API                                             | Behavior                                                                                                                                                                                  |
| ----------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `camera.getPicture(opts)`                       | Presents a **mobilealert** with “Camera / Photo Library” choices (unless `opts.type` provided), then acquires a file URI via Cordova Camera *or* a custom `modules.mediapicker` fallback. |
| `camera.get(opts, cb, err)`                     | Low-level getter that chooses Camera vs. Mediapicker based on `sourceType` and platform.                                                                                                  |
| `camera.uploadimg(imageURI, opts, fileobj, cb)` | Cordova FileTransfer upload to `app.uploadurl+'/upload/image/submit'`, with progress callback; returns `{ path, ext, ar, v }` or `false`.                                                 |

**Web fallback:** shows “web add event!” / returns false where native features aren’t available. 

***

## Orientation

| API                               | Behavior                                                                                |
| --------------------------------- | --------------------------------------------------------------------------------------- |
| `orientation.init()`              | Binds `orientationchange` and immediately locks to `'portrait-primary'` (Cordova only). |
| `orientation.lock()` / `unlock()` | Lock/unlock screen orientation if supported.                                            |

***

## Status Bar

| API                                      | Behavior                                                                                                                |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `statusBar.set(theme?)`                  | Apply **dark** or **light** content; updates `.iosbar` color for web preview too. Avoids redundant calls via `current`. |
| `statusBar.show(load?)` / `hide()`       | Show/hide status bar; Android has a retry loop to ensure overlay.                                                       |
| `statusBar.getHeight()` / `getCurrent()` | Helpers for layout and inspection.                                                                                      |

**Device CSS classes:** `device_{iOS|Android}`, `version_{major}`, `hasNotch`, `hasBottomNotch`, `iphoneX`. 

***

## Push Notifications

| API                                        | Behavior                                                                                                                                                                                                                                  |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `push.init()`                              | Initializes `PushNotification` plugin (Android channel creation, iOS categories), binds handlers for `registration`, `notification`, and custom actions (`accept/reject`), and registers device with backend (`/user/push/registerpush`). |
| `push.onNotification(e)`                   | Shows in-app toast when foreground and not currently viewing target chat; navigates on click or when background. Handles VoIP/call action separately.                                                                                     |
| `push.registerDevice(regid, isAndroid)`    | Sends device metadata + sandbox flag; caches `deviceid`/subscription.                                                                                                                                                                     |
| `push.setBadge(count)` / `clearBadge()`    | iOS badge number helpers.                                                                                                                                                                                                                 |
| `push.ensureChannel()` / `createChannel()` | Android notification channel management.                                                                                                                                                                                                  |

**Click-through routing**: uses `messagedata.route` to navigate via `app.history.go`. 

***

## Local Notifications (on-device scheduling)

| API                                          | Behavior                                                                                                                                       |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `localnotification.register(opts, data)`     | Queue-managed scheduling via `cordova.plugins.notification.local.schedule`; stores metadata in localStorage (`localnotification`). Returns id. |
| `localnotification.clear(id)` / `clearAll()` | Queue-managed clear operations.                                                                                                                |
| `localnotification.bind()`                   | Sets up `trigger` / `click` handlers and handles cold-start (`launchDetails`).                                                                 |
| `localnotification.handleNotification(id)`   | Optional deeplink: navigates to `data.route` if present.                                                                                       |

**Queue**: uses `async.queue` to serialize plugin calls reliably. 

***

## Footer Bar (iPhone X safe-area helper)

| API                                          | Behavior                                                                                                         |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `footerBar.init()`                           | (Currently early-returned/off) Would render `phone_footer` template and manage a bottom safe-area color overlay. |
| `footerBar.setColor(color)` / `getCurrent()` | Animate overlay color.                                                                                           |

***

## Device capabilities helpers

| API                                               | Behavior                                                                         |
| ------------------------------------------------- | -------------------------------------------------------------------------------- |
| `tapToPay.canUse()`                               | Cordova-only guard; iOS requires **≥ 16.4**; Android: `true`; web shows a toast. |
| `hasNotch()` / `hasBottomNotch()` / `isIphoneX()` | Heuristics for device cutouts and iPhone X class; adds body classes in `init()`. |

***

## Analytics

| API                | Behavior                                                                                                                              |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| `startAmplitude()` | Initializes Amplitude with `defaultTracking`; identifies with `app.user.getId()` when available; stores `phone.amplitudeInitialized`. |

***

## Lifecycle & system wiring (inside `phone.init()`)

* Sets `app.parseURL = phone.parseURL` and optionally reroutes using `?path=...`.
* Binds `document` listeners: `resume` (debounced), `pause`, `backbutton` (tries component `goBack()`, otherwise exits app).
* Detects device & version; toggles CSS classes and iOS RTC/adapter script for WebRTC support.
* Adjusts StatusBar visibility/overlay; detects notch and tablet widths; logs screen info.
* Initializes `footerBar`, `keyboard_global`, `bg_upload`, and **Mobiscroll** theme per platform. 

***

## Usage examples

### 1. Add/update an event in the user’s device calendar

```js
// Ensure user connects a default calendar first (one-time)
phone.calendar.linkCalendar(function (cal) {
  if (!cal) return; // user declined
  // Upsert an event
  phone.calendar.editEvent({
    id: 'evt_123',
    title: 'Community Gathering',
    eventLocation: 'Main Hall',
    notes: 'Bring snacks!',
    startDate: new Date('2025-10-12T18:00:00'),
    endDate: new Date('2025-10-12T19:30:00')
  }, 1, function () { console.log('Synced'); });
});
```

### 2. Get a photo & upload it

```js
phone.camera.getPicture({
  type: 'both',
  cb(imageURI) {
    phone.camera.uploadimg(imageURI, {
      data: { sizes: ['800x800','400x400'] },
      onProgress(p) { $('.progress').css('width', p + '%'); }
    }, null, function (resp) {
      if (!resp) return modules.toast({ content: 'Upload failed' });
      console.log('Uploaded to', resp.path);
    });
  },
  onExit() { console.log('User canceled'); }
});
```

### 3. Show dynamic app-icon badge with unread counts

```js
phone.badge.init({
  notificationEle: $('.notif-count'),
  notificationBadge: $('.notif-badge'),
  chatEle: $('.chat-count'),
  chatBadge: $('.chat-badge')
});
// Later, on entering a chat:
phone.badge.removeChat('room_42');
```

***

## Considerations & hardening (for all devices)

* **Cordova presence:** Many APIs no-op on web. Always guard flows that require native plugins and provide UX fallbacks.
* **Calendar IDs & permissions:** Selection is persisted; provide a way to **clear** default (`clearDefaultCalendar`) and re-prompt.
* **Error paths:** Some branches swallow errors (e.g., calendar ops, GPS failures). For critical flows, add callbacks/toasts.
* **Timezones:** Calendar dates are native `Date` objects; if events cross DST/timezones, ensure server and device are aligned.
* **Security:** Upload endpoints rely on `app.uploadurl`; ensure tokens/ACLs are enforced server-side.
* **Notch/safe areas:** The code already adds classes; use them in CSS to avoid content underlaps.
