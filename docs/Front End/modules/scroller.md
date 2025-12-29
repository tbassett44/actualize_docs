---
title: Scroller
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
# Module: `modules.scroller` — touch scrolling, snap, sticky headers, and “swipe-back” view integration

## High-level summary

A small, device-aware scroller that runs in either **native scroll** or **iScroll** mode, adds quality-of-life helpers (caret-aware auto-scroll while typing, keyboard insets, sticky headers), optional **horizontal** behavior with **snap-to** cards, and—importantly—**ties into your View Manager** so a **right-swipe** can drive a *“go back”* transition (`context.setX(dx)` while swiping, `context.goBack()` on release). 

***

## Constructor

```js
const sc = new modules.scroller(ele, options?, bindings?)
```

* `ele` — jQuery element acting as the scroll container (its **first child** is treated as the scrollable content).
* `options` — behavior flags (below).
* `bindings` — event callbacks (below). 

***

## Key options (most used)

| Option                 | Type                                    | Default | What it does                                                                                                                                                     |
| ---------------------- | --------------------------------------- | ------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type`                 | `'iscroll' \\| undefined`               |       — | If `'iscroll'`, initialize **IScroll** with `opts`; otherwise use native scrolling and custom handlers.                                                          |
| `scrollX`              | `bool`                                  | `false` | Horizontal mode; adds `scrollX` class; stores horizontal `max`.                                                                                                  |
| `passthrough`          | `bool`                                  | `false` | When container has nested `.scrollX`, temporarily sets `app.pauseSwipe=true` during inner scroll to avoid triggering **swipe-back**.                             |
| `noSwipe`              | `bool`                                  | `false` | Disable swipe gesture handling entirely (useful for modal content).                                                                                              |
| `context`              | `object`                                |       — | **View Manager integration**: on swipe, calls `context.setX(dx)`; on release with `dx>50`, calls `context.goBack()`. Also persists `scrollY` in `context.store`. |
| `swipeContainer`       | `jQuery`                                |       — | Legacy path for translating a container during swipe; current code delegates to `context.setX`.                                                                  |
| `snapTo`               | `selector`                              |       — | Horizontal **card snapping** target; computes nearest card and animates to it on touchend.                                                                       |
| `followTyping`         | `jQuery` (textarea/input)               |       — | Tracks caret and auto-scrolls so the cursor stays above the keyboard while typing.                                                                               |
| `hideKeyboardOnScroll` | `bool`                                  | `false` | On fast upward scroll (iOS/PG), hides the keyboard via `modules.keyboard_global.hide()`.                                                                         |
| `stickey`              | `{ ele, container, renderTo, context }` |       — | (Spelled **stickey** in code) Enables sticky header via **Waypoint**; clones element into `renderTo` when scrolled past. Offset honors notches.                  |
| `stickeyOffset`        | `number`                                |     `0` | Extra pixels added to sticky offset (notch/toolbar tuning).                                                                                                      |
| `hasInput / hasCursor` | `bool`                                  |       — | Enables caret-safe class toggling (`force-redraw`, `disableCaret`) and sets `probeType=3` for iScroll.                                                           |
| `resetScroll`          | `bool`                                  | `false` | If false and `context.store.scrollY` exists, restores previous scroll position.                                                                                  |

> Misc: `preventDefault` auto-relaxed when `window.forcemobile`; special casing for iPhoneX/bottom notch inside sticky offset. 

***

## Event bindings (3rd arg)

Provide any of these in the `bindings` object:

* **Swipe lifecycle:** `onSwipeStart(info)`, `onSwipe(info)`, `onSwipeEnd(info)` – Only fired when **not** paused and no text selection. Drives `context.setX`/`goBack` by default. 
* **Scroll lifecycle:** `scrollStart(obj)`, `scroll(obj)`, `scrollEnd(obj)`, `bounce(obj)` – `obj` includes current position, max, direction flags, and `bouncing` clamp state. Throttled on mobile. 
* **Resize:** `onResize()` – Triggered when content height changes (native mode via `ResizeSensor`; iScroll mode calls `refresh()`). 
* **Snap hooks:** `onSnapStart()`, `onSnapEnd({x})` – Around horizontal snap animation. 

***

## Public methods (instance)

| Method                                                               | Signature              | Notes                                                                                                                |                                                   |
| -------------------------------------------------------------------- | ---------------------- | -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| `getContainer()` / `getScroller()`                                   | \`() => jQuery         | IScroll\`                                                                                                            | Returns underlying container / scroller instance. |
| `getScrollContainer()`                                               | `() => jQuery`         | The **first child** of `ele` (content node).                                                                         |                                                   |
| `getOffset()` / `getHeight()` / `getContainerHeight()`               | —                      | Layout helpers.                                                                                                      |                                                   |
| `getNearest(opts, nearto)`                                           | `(opts, left) => left` | Computes snap target based on card widths and visible window.                                                        |                                                   |
| `setPosition(tele, editableContainer?)`                              | —                      | Cursor-aware auto-scroll (inputs, textareas, contenteditable). Accounts for keyboard height and optional “Done” bar. |                                                   |
| `bindInputs(iele)`                                                   | —                      | Wires focus/input/keyup to keep the caret visible; shows a **web keyboard** overlay in dev/web.                      |                                                   |
| `scrollTop(set?)` / `scrollToTop(delay?)` / `scrollToBottom(delay?)` | —                      | Works with both native and iScroll paths.                                                                            |                                                   |
| `scrollToElement(element, delay?, yoffset?)`                         | —                      | Scrolls so `element` top aligns (minus optional offset).                                                             |                                                   |
| `scrollBy(dy, time, debug?)`                                         | —                      | Positive `dy` moves content **down** (accounts for native vs. iScroll polarity).                                     |                                                   |
| `scrollTo({x?, y?, time?, ele?})`                                    | —                      | Scroll to absolute positions or to an element’s relative offset.                                                     |                                                   |
| `refresh()`                                                          | —                      | Recompute bounds; call `.refresh()` in iScroll mode.                                                                 |                                                   |
| `enable()` / `disable()`                                             | —                      | Toggle a simple `data('disabled')` guard checked in the scroll handler.                                              |                                                   |
| `destroy()`                                                          | —                      | If iScroll, call its `destroy()`.                                                                                    |                                                   |
| `ensure()`                                                           | —                      | Adjusts native scrollTop using last measured value (helper).                                                         |                                                   |
| `getStickyOffset()`                                                  | —                      | Computes header offset with notch-aware defaults and optional `stickeyOffset`.                                       |                                                   |

**Special events on container**\
The scroller listens for a custom **`inline_search`** event to bring the current active input into view (textarea path uses caret height). 

***

## View Manager (“swipe-back”) integration

* While the user swipes right (dx > 0), **and** `!app.pauseSwipe && !phone.isSelectingText`, the scroller calls:

  * `options.context.setX(e.dx)` – to translate the current view based on swipe distance.
  * On release, if `dx > 50`:

    * `options.context.goBack()` – to finalize the back navigation.
  * Otherwise, `options.context.setX(0)` – to cancel and snap back.
* You can **disable** this entirely via `noSwipe: true`, or temporarily by setting `app.pauseSwipe=true` (e.g., when interacting with a nested horizontal scroller). 

***

## Horizontal + snap-to cards

* Add `scrollX: true` and `snapTo: '.card'`.
* On touch end, the module measures each card’s left offset and **leans** toward the next/prev card using a 40/60% bias, then animates to that card using GSAP’s `scrollTo`.
* Hooks `onSnapStart/onSnapEnd` fire around the animation. 

***

## Sticky headers

Provide:

```js
options.stickey = {
  ele: $('.section-header:first'),
  container: $('.list'),
  renderTo: $('.sticky-slot'),
  context   // view context (for rendering via phi.render)
}
```

A **Waypoint** pins/clones the header into `renderTo` when scrolled past; `getStickyOffset()` accounts for iOS status-bar/notch height and `stickeyOffset`. 

***

## Keyboard & caret UX

* `followTyping`: when set, caret movement in a textarea/input triggers auto-scroll so the cursor stays above the keyboard.
* `bindInputs(...)` also shows a **Done** button row (`keyboard_done` template) when inputs opt-in via `data-showdone`, and removes it when the keyboard hides.
* Fast upward scroll with `hideKeyboardOnScroll: true` will programmatically close the keyboard on mobile (threshold: speed \< −2). 

***

## Usage examples

### 1. Standard vertical feed with swipe-back

```js
const sc = new modules.scroller($('.feed'), {
  context: viewManager,           // must implement setX(dx) and goBack()
  followTyping: $('.composer textarea'),
  hideKeyboardOnScroll: true
}, {
  scrollStart(o){ /* maybe fade a shadow */ },
  scrollEnd(o){ /* load more if near bottom */ }
});
```

### 2. Horizontal cards with snapping

```js
new modules.scroller($('.carousel'), {
  scrollX: true,
  snapTo: '.card',
  passthrough: true // nested scrolls don’t trigger view swipe-back
}, {
  onSnapEnd({x}){ console.log('snapped to', x); }
});
```

### 3. Sticky section header

```js
new modules.scroller($('.list'), {
  stickey: {
    ele: $('.section-header'), container: $('.list'),
    renderTo: $('.sticky-slot'), context
  }
});
```

***

## Things to watch (and small hardening ideas)

* **Option name typo:** the sticky option is spelled **`stickey`** in code. Document it as-is, or add a compatibility alias. 
* **Back-swipe threshold:** `dx>50` is hard-coded; consider exposing `backSwipeThreshold`. 
* **Performance:** In horizontal lists with many cards, `getNearest` queries all matches each gesture—cache widths/offsets if needed. 
* **Keyboard math:** caret calculations rely on `textareaHelper('caretPos')` and `modules.keyboard_global.keyboardHeight`; ensure both are present in every screen where `followTyping` is used. 
* **Selection guard:** Swipes are ignored while selecting text (`phone.isSelectingText`). If users report missed swipes, check for unintended selection states. 
* **Waypoints cleanup:** If you build/destroy lists frequently, consider exposing a `teardown()` that also removes Waypoints held in `self.stickeys`.
