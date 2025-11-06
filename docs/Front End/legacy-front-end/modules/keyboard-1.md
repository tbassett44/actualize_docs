---
title: Keyboard
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
# Keyboard Module (JS + Templates) — Developer Docs

## High-level overview

The keyboard module manages showing/hiding the on-screen keyboard and automatically adjusts your UI: it animates your input panel up by the keyboard height, resizes any scrollable area, and exposes lifecycle hooks (will/did show/hide). It also ships a debug “web keyboard” overlay to simulate keyboard behavior in desktop browsers using the `keyboard_web` template.  

This often works in tandem with the [Infinitescroll] module

***

## Public APIs

### `modules.keyboard(options)`

Creates a keyboard controller bound to a specific UI container.

**Common options (inferred from usage in code):**

| Option                                      | Type                 | Purpose                                                                                                                 |
| ------------------------------------------- | -------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `ele`                                       | jQuery element       | Root element whose bottom is animated when the keyboard shows/hides.                                                    |
| `scrollele`                                 | jQuery element       | A scrollable content element; its bottom inset is adjusted by `keyboardHeight + headerHeight` so content stays visible. |
| `room`                                      | string               | Logical channel/room used for socket wiring and message state (start/stop sockets).                                     |
| `keyboardOptions.adjustOnKeyboardShow`      | boolean              | If true, scrolls by the keyboard/header delta on show.                                                                  |
| `onKeyboardWillShow` / `onKeyboardWillHide` | function             | Per-instance callbacks fired when the OS keyboard is about to appear/disappear.                                         |
| `identity`                                  | object               | Overrides the active “sender identity” used for read receipts, etc. Falls back to current user.                         |
| `disableSockets`                            | boolean              | When true, skips socket start/stop wiring.                                                                              |
| `showDiff`                                  | number               | Subtracts an additional offset from reported keyboard height (useful for safe-area/notch tuning).                       |
| `cache`                                     | \{ `until`: Number } | Enables message draft caching under `options.room` key.                                                                 |

**Key methods (instance):**

* `start()` / `stop()` — attach/detach global keyboard hooks; start/stop sockets if enabled. On stop, it also resets the panel position. 
* `hide()` — programmatically dismiss the OS keyboard (Cordova/Capacitor: `Keyboard.hide/close`; web: blur active element). Also hides the debug web keyboard if active. 
* `onKeyboardWillShow(height, time)` — internal: animates `ele` up by `height`, adjusts `scrollele`, refreshes scroller, and caps text area max height to available space. (Use global hooks below to plug in.) 
* `onKeyboardWillHide(time)` — internal: animates `ele` back to bottom; restores `scrollele` bottom inset to header height. 
* `setAvailableHeight()` — computes how tall the composer textarea may grow, based on viewport minus keyboard and header, with upper bound of \~200px. 
* `cacheMessage(clear)` — stores or clears the current draft in `modules.exp_prefs` when `options.cache` is provided. 

**Behavioral details:**

* When the keyboard appears, the module:

  * Animates the composer container (`ele`) to `bottom: keyboardHeight`. 
  * Offsets the scroller’s bottom by `keyboardHeight + headerHeight` (with a small iOS notch correction) and optionally scrolls content. 
  * Recomputes “available height” for expanding textareas. 
* When the keyboard hides, both `ele` and `scrollele` are animated back, and the scroller is refreshed. 

***

### `modules.keyboard_global`

A singleton that listens to native (or simulated) keyboard events and fans them out to your `modules.keyboard` instances or your own handlers.

**Useful properties & methods:**

| API                                                                                     | Type     | Purpose                                                                                                                             |                                                                                                                                        |
| :-------------------------------------------------------------------------------------- | :------- | :---------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------- |
| `onKeyboardWillShow` / `onKeyboardDidShow` / `onKeyboardWillHide` / `onKeyboardDidHide` | function | false                                                                                                                               | Assign per-app callbacks to react to lifecycle events. They receive normalized keyboard height (in px). Debounced via `throttleEvent`. |
| `oneTimeKeyboardWillShow` / `oneTimeKeyboardOnShow` / `oneTimeKeyboardWillHide`         | function | false                                                                                                                               | Fire a handler only once on the next event.                                                                                            |
| `overrides`                                                                             | object   | false                                                                                                                               | If set, uses these override handlers instead of the standard ones (useful for temporary modal flows).                                  |
| `keyboardHeight`                                                                        | number   | The most recently observed keyboard height.                                                                                         |                                                                                                                                        |
| `hasWebKeyboard()`                                                                      | function | Returns whether the debug web keyboard is available.                                                                                |                                                                                                                                        |
| `showWebKeyboard()` / `hideWebKeyboard()`                                               | function | Shows/hides the debug “web keyboard” overlay using the `keyboard_web` template and animates it in/out; also fires lifecycle events. |                                                                                                                                        |
| `hide()`                                                                                | function | Cross-platform programmatic hide (Cordova/Capacitor and web).                                                                       |                                                                                                                                        |

**Global event fan-out (what it does):**

* Normalizes native event payloads into a consistent `keyboardHeight`.
* Triggers override handlers if present; otherwise, calls one-time handlers, then persistent handlers (`onKeyboardWillShow/DidShow/WillHide/DidHide`).
* Notifies `phi` instrumentation hooks (`phi.onKeyboardWillShow/DidShow/WillHide/DidHide`) for broader app analytics/rendering. 

***

## Templates

* `keyboard_web` — used by `modules.keyboard_global.showWebKeyboard()` to simulate a keyboard in desktop environments for layout testing. The overlay animates from the bottom and drives the same event flow as a real keyboard.\
  *(Other keyboard templates were uploaded separately; this doc references only templates that appear in the JS and are accessible.)*

***

## Integration notes & caveats

* **Safe-area quirks (iOS notch):** On iOS with a bottom notch, the module subtracts \~20px to avoid over-shifting content. If your device padding differs, adjust via `showDiff`.  
* **Draft caching:** If you provide `options.cache`, the module stores the current input under the `options.room` key with an expiry (`cache.until`). Clear it via `cacheMessage(true)`. 
* **Scroll coordination:** Provide `scrollele` if you want automatic inset and scroll-by behavior on show. Without it, only the composer panel (`ele`) will animate. 
* **Sockets (optional):** If `disableSockets` is falsey, `start()`/`stop()` wire to `app.user.startSocket/stopSocket(room, onMessage)` for chat-style use cases. If you don’t use sockets, set `disableSockets: true`. 

***

## Minimal example

```js
// Create a controller for your chat composer
const kb = new modules.keyboard({
  ele: $('.chat-composer'),         // the input panel to move
  scrollele: $('.chat-scroll'),     // the scrollable feed
  room: 'room:general',             // optional socket key
  keyboardOptions: { adjustOnKeyboardShow: true },
  onKeyboardWillShow() { /* e.g., pause videos */ },
  onKeyboardWillHide() { /* e.g., resume videos */ },
  cache: { until: 60 * 60 }         // seconds (example)
});

// Start listening
kb.start();

// Later…
kb.hide();   // programmatically close the keyboard
kb.stop();   // unbind + reset
```
