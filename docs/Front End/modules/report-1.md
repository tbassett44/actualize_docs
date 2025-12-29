---
title: Report
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
# Module: `modules.report` — Report sheet (reason + optional “immediate” flag) modal

> A lightweight, animated “report content” sheet that collects a free-text reason (autosizing textarea) and an optional **Immediate attention** toggle, then submits everything to your server. It supports swipe-down to dismiss, keyboard-safe insets, custom template/target element, and success/error toasts. 

***

## What it renders & how it behaves (at a glance)

* **Modal sheet** injected into `renderTo` (defaults to `body`) using `template` (defaults to `report_page`).
* **Controls**:

  * Close (`.x_closer`) → dismiss with slide-down.
  * Send (`.x_send`) → validates non-empty reason, shows a spinner, POSTs, toasts result.
  * Immediate toggle (`.immediate`) → adds/removes `.toggled` and sets `options.immediate` to `1` or empty.
  * Link (`.x_link`) → opens community standards page.
* **Textarea** (`.autosize` + `textarea`) grows with input and mirrors into `options.report`.
* **Gestures**: drag the sheet (`.swiper`) \~>80px to dismiss (GSAP Draggable).
* **Keyboard-safe**: when the keyboard shows, the content’s bottom inset animates to keyboard height; reset on hide.
* **Animations**: sheet slides from 100% to 0% Y, dimmed backdrop fades in/out. 

***

## Public API

### `new modules.report(options)`

Creates a report sheet instance. Call `show()` to render.

#### Options (incoming)

| Option      | Type           |            Default | Purpose                                                                                |
| ----------- | -------------- | -----------------: | -------------------------------------------------------------------------------------- |
| `renderTo`  | jQuery element |        `$('body')` | Where to render the modal.                                                             |
| `template`  | string         |    `'report_page'` | Template key expected to include the required hooks/classes (see below).               |
| `title`     | string         | `'Report Content'` | Title merged into template data.                                                       |
| `report`    | string         |                  — | Initial reason text (kept in sync with textarea).                                      |
| `immediate` | `1 \\| ''`     |               `''` | Urgency flag toggled via `.immediate`.                                                 |
| `…`         | any            |                  — | **Passed through untouched** to the API (e.g., `subjectId`, `subjectType`, `context`). |

> All fields in `options` are submitted as the POST body—use this to attach metadata about what’s being reported. 

#### Required template hooks (CSS classes inside your template)

* `.x_closer`, `.x_send`, `.immediate`, `.x_link`
* `.autosize` and a `textarea`
* `.pane` (the sliding panel), `.swiper` (drag handle area), wrapper element for backdrop & `.content` (for keyboard inset)\
  These are directly referenced by the module’s event bindings/animations. 

***

### Instance methods

| Method      | What it does                                                                                                                                                                                             |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `show()`    | Renders the modal; binds handlers; enables drag-to-dismiss; sets keyboard overrides; animates in.                                                                                                        |
| `send()`    | Validates that `options.report` is non-empty; swaps the send button’s text with a spinner; POSTs to `app.sapiurl + '/module/report/send'`; restores button text; toasts success/error; hides on success. |
| `hide()`    | Hides keyboard; animates backdrop/pane out; calls `destroy()` after animation.                                                                                                                           |
| `destroy()` | Clears keyboard overrides, kills the Draggable instance, removes the element, frees the instance.                                                                                                        |

> Networking uses your existing `modules.api` wrapper. Success path shows *“Successfully Submitted Report!”*; failure toasts *“Error Saving: …”*. 

***

## Usage example

```js
const report = new modules.report({
  title: 'Report this post',
  subjectId: 'post_123',
  subjectType: 'post',
  renderTo: $('body'),
  // prefill example:
  report: '',
  immediate: '' // or 1
});

// Present the sheet
report.show();
```

**Template tips:** ensure your `report_page` (or custom) template includes the hooks above; the textarea should have class `.autosize` so it grows with input.

***

## Integration details & dependencies

* **UI/Anim**: GSAP TweenLite + Draggable (optional; module checks `window.Draggable`), `.autosize()` plugin for the textarea.
* **Keyboard**: `modules.keyboard_global.overrides` adjust `.content` bottom inset on keyboard show/hide; cleared on destroy.
* **Toasts**: `modules.toast(…)` for validation and result messages.
* **HTTP**: `modules.api` posts to `app.sapiurl + '/module/report/send'` with the **full`options` object**.
* **Link**: `_.openLink({ intent: 'https://actualize.earth/community-standards' })` for community guidelines. 

***

## Validation & UX flow

1. User types a reason → mirrored to `options.report`.
2. Taps **Send** → if empty, a toast asks for a reason; otherwise the button shows a spinner and the request is sent.
3. On success → success toast + dismiss; on error → error toast (button label restored both paths). 

***

## Considerations & suggested hardening (poking holes)

* **Server contract**: The module submits the **entire`options` object**; document your backend’s expected fields (e.g., `subjectId`, `subjectType`, `report`, `immediate`, `reportedBy`). Consider whitelisting fields on the client before sending to avoid leaking UI-only flags. 
* **Empty reason only check**: It only checks *existence* and *length*; you might want to trim whitespace and enforce a minimum length (e.g., ≥ 10 chars). 
* **Keyboard override scope**: Overrides are global; if multiple components rely on `modules.keyboard_global`, ensure the override is reset even on exceptional exits (route change). The module does this in `destroy()`, but also consider guarding route transitions. 
* **Drag threshold**: Dismiss threshold is fixed at `endY > 80`; consider exposing an option (e.g., `dismissDistance`) for accessibility (reduced motion / tremor). 
* **Accessibility**: Add ARIA roles (`dialog`), focus trapping, and restore focus to the invoker on close.
* **Internationalization**: Button labels, toasts, and link copy are currently hard-coded in English; pass them in via `options` or use your i18n layer.
* **Error messages**: Surface more context on server errors (status codes, retry advice).
* **Rate limiting**: Prevent rapid multiple submissions (disable send button while in flight—partly covered by spinner, but not a strict guard).
* **Analytics**: Consider emitting an event on open/submit/cancel for abuse monitoring funnels.

***

## Minimal extension points

* **Custom endpoint or payload**\
  If you need a different endpoint, wrap `send()` or add `options.endpoint` and change the URL accordingly before calling `modules.api`.
* **Pre-submit sanitizer**\
  Add a hook to normalize payload (trim reason, clamp length, redact PII).
* **Theming**\
  Allow `options.theme` to tweak background opacity, corner radius, etc., when rendering.
