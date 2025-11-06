---
title: Alert
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
<div id="test"></div>
<script>document.querySelectorAll('#test').innerHTML('TEST')</script>

<br />

# Module: `$.fn.alert` (jQuery modal/alert plugin)

## High-Level Summary

This module defines a jQuery plugin `$.fn.alert(obj)` that renders a modal-style alert/notification with configurable content, appearance, behavior, and lifecycle callbacks. It supports queuing multiple modals (`storedmodal`) when one is already open, animated open/close, optional template-based rendering, auto-sizing, keyboard/ESC closing, blur-the-background effects, and a button bar. The plugin centralizes z-index management to keep the most recent alert on top and exposes helpers like `$.fn.alert.getAlert()` to retrieve the topmost modal. Integration points include a render engine (`$(el).render(opts)` with templates), a custom scroller, and GSAP/TweenLite for animations.

## Options / Props

> Unless noted, all options are **optional**. Types are inferred from usage.

| Name             | Type                                       | Required | Default                                    | Accepted Values / Shape                                          | Description                                                                                     |
| ---------------- | ------------------------------------------ | -------: | ------------------------------------------ | ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `id`             | `string`                                   |       No | `"alert"`                                  | —                                                                | Instance identifier.                                                                            |
| `uid`            | `string`                                   |       No | `"alert"`                                  | —                                                                | Used in templating and element bindings.                                                        |
| `animatedmodal`  | `boolean`                                  |       No | `true`                                     | —                                                                | Hint that modal is animated (not directly referenced elsewhere).                                |
| `minimizable`    | `boolean`                                  |       No | `false`                                    | —                                                                | If true, enables minify/expand behaviors (class toggling provided).                             |
| `icon`           | `string`                                   |       No | `""`                                       | —                                                                | Icon name/HTML; not directly used in core logic.                                                |
| `fixed`          | `boolean`                                  |       No | `$.fn.alert.fixed ? true : false`          | —                                                                | If truthy, treats the alert as fixed-position (consumed by template/CSS).                       |
| `iconcolor`      | `string`                                   |       No | `"white"`                                  | —                                                                | Icon color (template/CSS concern).                                                              |
| `closerColor`    | `string`                                   |       No | `"white"`                                  | —                                                                | Close button color (template/CSS concern).                                                      |
| `title`          | `string`                                   |       No | `""`                                       | —                                                                | Modal title.                                                                                    |
| `titleColor`     | `string`                                   |       No | `""`                                       | —                                                                | Title color (template/CSS concern).                                                             |
| `timeout`        | `number`                                   |       No | `1500`                                     | ms                                                               | Available for consumers/templates to auto-close.                                                |
| `mobilize`       | `boolean`                                  |       No | `false`                                    | —                                                                | Mobile mode toggle (prevents ESC-close overlay binding).                                        |
| `rele`           | `object \| false`                          |       No | `false`                                    | `{ render(obj) }`                                                | If truthy, delegates to `rele.render(obj)` and returns early.                                   |
| `close_above`    | `boolean`                                  |       No | `false`                                    | —                                                                | Template/behavior hook (not used in core).                                                      |
| `clearAnimation` | `boolean`                                  |       No | `false`                                    | —                                                                | Template/behavior hook.                                                                         |
| `hideonkeyboard` | `boolean`                                  |       No | `false`                                    | —                                                                | Template/behavior hook.                                                                         |
| `overflow`       | `boolean`                                  |       No | `false`                                    | —                                                                | Template/behavior hook; actual overflow set by `setAutoHeight`.                                 |
| `image`          | `boolean \| string`                        |       No | `false`                                    | `false` or HTML string                                           | If `false` disables image; otherwise can be HTML/URL used by template.                          |
| `module`         | `boolean`                                  |       No | `false`                                    | —                                                                | Template/behavior hook.                                                                         |
| `modalClass`     | `string`                                   |       No | `""`                                       | —                                                                | Additional class for modal root.                                                                |
| `background`     | `string`                                   |       No | `"black"`                                  | —                                                                | Modal background color (template/CSS).                                                          |
| `color`          | `string`                                   |       No | `"white"`                                  | —                                                                | Text color (template/CSS).                                                                      |
| `width`          | `number`                                   |       No | `400`                                      | px                                                               | Base content width (overridden by auto width logic).                                            |
| `zIndex`         | `number`                                   |       No | `$.fn.alert.alertzindex`                   | —                                                                | z-index assigned to this alert; auto-incremented globally.                                      |
| `animate`        | `boolean`                                  |       No | `true`                                     | —                                                                | Enables open/close animations.                                                                  |
| `closer`         | `boolean`                                  |       No | `true`                                     | —                                                                | Whether a close button is rendered.                                                             |
| `autowidth`      | `boolean`                                  |       No | `false`                                    | —                                                                | If true, `setAutoWidth` sets content width based on viewport.                                   |
| `autoheight`     | `boolean`                                  |       No | `false`                                    | —                                                                | If true, `setAutoHeight` constrains content height to viewport.                                 |
| `textsize`       | `number`                                   |       No | `14`                                       | px                                                               | Base text size (template/CSS).                                                                  |
| `iconsize`       | `number`                                   |       No | `60`                                       | px                                                               | Icon size (template/CSS).                                                                       |
| `spacing`        | `number`                                   |       No | `45`                                       | px                                                               | Vertical padding used when computing available height.                                          |
| `maxwidth`       | `number`                                   |       No | `900`                                      | px                                                               | Used by `setAutoWidth` as `mw`.                                                                 |
| `maxHeight`      | `boolean \| number`                        |       No | `false`                                    | —                                                                | If truthy, forces content to available height.                                                  |
| `escClose`       | `boolean`                                  |       No | `false`                                    | —                                                                | If true (and not `mobilize`), clicking `.clickclose` closes the modal.                          |
| `buttons`        | `Array<{ btext: string, bclass: string }>` |       No | `[ { btext:'OK', bclass:'x_closer' } ]`    | —                                                                | Button config; `.x_closer` is wired to close.                                                   |
| `template`       | `string`                                   |       No | uses `$.fn.alert.useTemplate` or `"alert"` | —                                                                | Template name for `$(el).render`.                                                               |
| `content`        | `string`                                   |       No | `""`                                       | —                                                                | Rendered inner HTML. If absent but `template` provided, the plugin renders via template engine. |
| `tempdata`       | `object`                                   |       No | `{}`                                       | —                                                                | Data passed into template render if `content` is not provided.                                  |
| `rendered`       | `number`                                   |       No | `0`                                        | `0` or `1`                                                       | Flag set when content has been rendered from a template.                                        |
| `preview`        | `function(show: () => void)`               |       No | —                                          | Hook to prepare content before showing; call `show()` to reveal. |                                                                                                 |
| `binding`        | `function(ele, data)`                      |       No | —                                          | Additional binding pushed into `opts.bindings`.                  |                                                                                                 |
| `bindings`       | `Array`                                    |       No | —                                          | Additional bindings appended to the default binding.             |                                                                                                 |
| `replace`        | `boolean`                                  |       No | `false`                                    | —                                                                | If false and a modal is open (and not `overlay`), the new modal is queued.                      |
| `overlay`        | `boolean`                                  |       No | `false`                                    | —                                                                | If true, allows multiple simultaneous modals (skip queue).                                      |
| `dontstore`      | `boolean`                                  |       No | `false`                                    | —                                                                | If true and a modal is already open, do not queue the new one.                                  |
| `blurEle`        | `jQuery`                                   |       No | `#wrapper` (lazy default)                  | —                                                                | Element to blur; removed from `opts.data` before rendering (stored later on `data`).            |
| `noblur`         | `boolean`                                  |       No | `false`                                    | —                                                                | Global/static toggle on `$.fn.alert` that bypasses blur.                                        |

## Events / Callbacks

| Name             | Params                                    | When It Fires                           | Notes                                                                                                            |
| ---------------- | ----------------------------------------- | --------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `triggerModal`   | none                                      | After closing, if queue has items       | Dequeues next stored modal.                                                                                      |
| `onResize`       | `(force:boolean, ele?:jQuery)`            | On external resize triggers             | Drives `setAutoWidth/Height`.                                                                                    |
| `closeAlert`     | `(e?:Event, ele?:jQuery, force?:boolean)` | When closing programmatically or via UI | Handles animation, callbacks, queue, z-index; also exported to `$.fn.alert.closeAlert`.                          |
| `setAutoWidth`   | `(ele?:jQuery)`                           | When `autowidth` true                   | Sets `.alertcontent` width to 90% of body width, capped by `maxwidth`.                                           |
| `setAutoHeight`  | `(ele?:jQuery, force?:boolean)`           | When `autoheight` true                  | Limits `.alertcontent` height to viewport minus `spacing*2`; sums `.calcheight:visible`. Calls `onHeightUpdate`. |
| `onHeightUpdate` | `()`                                      | After autoheight recalculation          | Override to respond to height changes.                                                                           |
| `clearBlur`      | `()`                                      | On close or minify/expand               | Removes blur classes from `blurEle`.                                                                             |
| `setBlur`        | `()`                                      | On open                                 | Adds blur classes to `blurEle` (skips if `noblur`/`obj.noblur`).                                                 |
| `minify`         | `()`                                      | On user action                          | Adds `.minify` to the alert root.                                                                                |
| `expand`         | `()`                                      | On user action                          | Removes `.minify`.                                                                                               |
| `onOpen`         | `(ele, data)`                             | After the modal is shown                | Post-open hook.                                                                                                  |
| `onClose`        | `(force?:boolean)`                        | After element removal                   | Post-close hook.                                                                                                 |

## Dependencies & Side Effects

- **Internal deps:** `$(this).render(opts)`, `$.fn.render.getTemplate(name).render(data)`, `modules.scroller(ele.find('.relativealert'))`
- **External deps:** GSAP/TweenLite, jQuery, a tap/gesture plugin providing `.stap`, global `isMobile`
- **DOM classes required by your template:** `.modalalert` (root), `.alertcontent` (for sizing); optional `.scrollingarea`, `.relativealert`, `.normalfastanimated`, `.clickclose`, `.calcheight`, `.x_closer`
- **Global state:** `$.fn.alert.alertzindex` (starts ~500), `$.fn.alert.storedmodal` (queue), `$.fn.alert.currentalertopts`, `$.fn.alert.noblur`
- **Queuing:** If an alert is open and `!replace && !overlay`, new alerts are queued (unless `dontstore`), with `animate=false`, to display after the current one closes.

## Option Details

### `rele`

- **Purpose:** Bypass plugin rendering and delegate to `rele.render(obj)`.
- **Side effects:** Sets `obj.append=false`, ensures `obj.data` exists, and forces `obj.data.groupselect=1`; copies `tempdata` into `data`. Returns early.

### `autowidth` / `maxwidth`

- **Logic:** `w = body.width() * 0.9`, then `min(w, maxwidth)`, applied to `.alertcontent`.
- **Note:** Code checks `data.maxWidth` (camelCase) in one branch but default is `maxwidth` (lowercase). **Standardize the key** (recommended: `maxWidth`).

### `autoheight` / `maxHeight` / `spacing`

- **Logic:** If `maxHeight` truthy, set `.alertcontent` height to `body.height() - spacing*2`. Else, sum `.calcheight:visible` heights and clamp if exceeding available space. Calls `onHeightUpdate`.

### `escClose`

- **Behavior:** If `true` and not `mobilize`, binds `.clickclose` overlay to trigger `closeAlert`.

### `template` / `content` / `tempdata`

- **Behavior:** If `content` missing but `template` provided, renders via `$.fn.render.getTemplate(template).render({ template, uid, _tid: uid, ...tempdata })`.

## Usage Example

```js
$('body').alert({
  title: 'Saved!',
  content: '<p>Your settings were saved successfully.</p>',
  animate: true,
  escClose: true,
  autowidth: true,
  autoheight: true,
  buttons: [
    { btext: 'OK', bclass: 'x_closer' }
  ],
  onOpen(ele, data) {
    ele.find('.x_closer').first().trigger('focus');
  },
  onClose() {
    console.log('Alert closed');
  }
});
```

```js
$('#app').alert({
  template: 'alert',
  tempdata: { title: 'Welcome', body: 'This is a template-rendered alert.' },
  autowidth: true,
  autoheight: true,
  maxwidth: 640,
  buttons: [
    { btext: 'Cancel', bclass: 'x_closer' },
    { btext: 'Continue', bclass: 'primary next-step' }
  ],
  binding(ele, data) {
    ele.on('click', '.next-step', () => data.closeAlert(null, ele));
  }
});
```

## Notes & Edge Cases

- **Z-index lifecycle:** Always close via `$.fn.alert.closeAlert()` to keep global z-index in sync.
- **Queued modals:** Use `dontstore: true` to skip queueing, or `overlay: true` to allow stacking.
- **Blur behavior:** Provide a valid `blurEle` (defaults to `#wrapper`) or set `noblur` to disable.
- **Mobile:** On close, if `isMobile`, scroll resets to `(0,0)`.
- **Animation classes:** Ensure CSS for `.normalfastanimated` with `zoomIn/zoomOut` and overlay fade.
- **`shouldClose` hook:** The code calls `obj.shouldClose()` if present and aborts close when it returns `false`. Add it if you need guardrails.
- **Potential bug:** `maxWidth` vs `maxwidth` mismatch. Standardize to one key.

## Exported Helpers

- `$.fn.alert.getAlert(): jQuery \| false` — returns the topmost `.modalalert` by z-index.
- `$.fn.alert.closeAlert` — function reference to instance `closeAlert`.
- `$.fn.alert.cdot` — base64 1×1 PNG dot (transparent placeholder).