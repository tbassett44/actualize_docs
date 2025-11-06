---
title: Code Picker
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
# Module: `modules.codepicker` (Modal code entry & validation)

## High-Level Summary

`modules.codepicker(options)` renders a modal “enter code” view with a title, input, and confirm/close actions. It can either (a) **POST the code to an API** via `modules.api` and invoke `onSuccess` on positive responses, or (b) **hand the code to a local `onSubmit` callback** when no endpoint is provided. It uppercases input keystrokes, animates in/out with GSAP/TweenLite, and prevents double-submissions with an internal `confirming` flag.

***

## Options / Props

> All options are optional unless marked **Yes** in Required.

| Name              | Type                               | Required | Default        | Description                                                                                                                                      |                                                    |
| ----------------- | ---------------------------------- | -------: | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------- |
| `submitBtn`       | `string`                           |       No | `"Confirm"`    | Button label shown on the confirm button.                                                                                                        |                                                    |
| `title`           | `string`                           |       No | `"Enter Code"` | Modal title (used by template).                                                                                                                  |                                                    |
| `endpointKey`     | `string`                           |       No | `"code"`       | The field name used in the request body to carry the code.                                                                                       |                                                    |
| `placeholderText` | `string`                           |       No | `"Enter Code"` | Input placeholder text.                                                                                                                          |                                                    |
| `endpoint`        | `string`                           |       No | —              | If provided, `confirm()` will call `modules.api({ url: endpoint, ... })`.                                                                        |                                                    |
| `endpointOpts`    | `object`                           |       No | —              | Merged into `data` sent to the endpoint.                                                                                                         |                                                    |
| `schema`          | \`string                           | object\` | No             | —                                                                                                                                                | If set, added to the API request as `data.schema`. |
| `onSuccess`       | `(resp, hidePicker, code) => void` |       No | —              | Called after a **successful** API response (`resp.success === true`). Receives the server `resp`, a `hidePicker` function, and the `code` value. |                                                    |
| `onSubmit`        | `(code, hidePicker) => void`       |       No | —              | Used when **no** `endpoint` is provided. Gets the `code` and a `hidePicker` function.                                                            |                                                    |
| `onHide`          | `() => void`                       |       No | —              | Not passed via options; assignable afterwards: `instance.onHide = fn`. Called when picker finishes hiding.                                       |                                                    |

> Internally, defaults are stored in `self.options = $.extend(true, defaults, options)`. Some code paths reference the original `options` object directly; see **Notes**.

***

## Events / Callbacks

| Name            | Params                     | When It Fires                         | Notes                                                                    |
| --------------- | -------------------------- | ------------------------------------- | ------------------------------------------------------------------------ |
| `confirm()`     | —                          | On confirm button tap                 | Validates state, shows spinner, collects input, calls API or `onSubmit`. |
| `onSuccess`     | `(resp, hidePicker, code)` | After API returns `{ success: true }` | Your hook to proceed (e.g., route, close modal).                         |
| `onSubmit`      | `(code, hidePicker)`       | When no `endpoint` is provided        | Synchronous/local submission path.                                       |
| `onHide`        | `()`                       | After hide animation completes        | Only if `instance.onHide` has been set externally.                       |
| Close button    | —                          | On `.x_close` tap                     | Triggers `hidePicker()`.                                                 |
| Uppercase input | —                          | On `.x_code` `keyup`                  | Forces uppercase for every keystroke.                                    |

***

## Dependencies & Side Effects

- **Rendering:** `$('body').render({ template: 'codepicker_page', data, binding })` (template must output specific class hooks, below).
- **Animation:** GSAP/TweenLite (`set` and `to`).
- **Touch/click:** `.stap(handler, 1, 'tapactive')` tap abstraction.
- **API:** `modules.api` for network calls; `modules.toast({ content })` for error messages.
- **Keyboard:** If present, `modules.keyboard_global.hide()` is called before showing.
- **DOM contract (template must provide):**

  - `.pane` (sliding drawer)
  - `.bg` (dim overlay)
  - `.x_close` (close button)
  - `.x_confirm` (confirm button)
  - `.x_code` (input field)

***

## Option Details

### Network submission vs. local submission

- If `options.endpoint` is provided, `confirm()` calls:

  - URL: `options.endpoint`
  - Body: `{ [endpointKey]: value, schema?, ...endpointOpts }`
  - Timeout: `40000ms`
  - On response: if `resp.success`, call `options.onSuccess(resp, hidePicker, code)`; else show `modules.toast({ content: resp.error })`.
- If **no** `endpoint`, `confirm()` calls `options.onSubmit(code, hidePicker)` if provided.

### Submission guard & button state

- Uses `this.confirming` to avoid duplicate submits.
- Sets `.x_confirm` HTML to a spinner (`<i class="icon-refresh animate-spin"></i>`) during request.
- Restores the button label to `options.submitBtn` after **API callback** returns.

### Show / Hide animations

- On show: slides `.pane` up from its height; fades `.bg` opacity to `0.3`.
- On hide: reverses animations and removes the element from DOM. Calls `instance.onHide()` if provided.

### Input normalization

- Forces **uppercase** on every `keyup` in `.x_code`.

***

## Usage Example

```js
// 1) Server-validated code entry
new modules.codepicker({
  title: 'Enter Invite Code',
  placeholderText: 'e.g., ABCD-1234',
  endpoint: app.apiurl + '/codes/verify',
  endpointKey: 'invite_code',
  endpointOpts: { campaign: 'launch' },
  schema: 'invite',
  onSuccess(resp, hidePicker, code) {
    // success UX (e.g., navigate, store token, etc.)
    console.log('Code OK:', code, resp);
    hidePicker(() => modules.toast({ content: 'Welcome!' }));
  }
});
```

```js
// 2) Local-only validation (no network)
const picker = new modules.codepicker({
  title: 'Enter Discount Code',
  onSubmit(code, hidePicker) {
    if (code === 'LOVE-2025') {
      hidePicker(() => modules.toast({ content: '15% discount applied' }));
    } else {
      modules.toast({ content: 'Invalid code' });
    }
  }
});
// Optional: react to hide
picker.onHide = () => console.log('Picker closed');
```

***

## Notes & Edge Cases (rigorous checks)

1. **Inconsistent options access (`options` vs `self.options`)**  
   Inside `confirm()`, the code mixes `options.*` and `self.options.*`. If you mutate `self.options` later, `confirm()` may still read the original `options`.  
   **Recommendation:** Use **only `self.options`** inside methods.

2. **Button label restoration when no endpoint**  
   The spinner is shown before branching, but if **no `endpoint`** is provided, the button label is **not restored** to `submitBtn`.  
   **Fix:** Reset `.x_confirm` text in the `else` branch after invoking `onSubmit`.

3. **`confirming` reset on non-endpoint path**  
   `this.confirming` is cleared only in the API callback. In the no-endpoint branch, it remains `true`, blocking subsequent confirms.  
   **Fix:** Set `self.confirming = false;` after `onSubmit()` returns.

4. **Uppercasing codes**  
   All input is forced to uppercase. If case matters for certain codes, this will break validation.  
   **Fix:** Make this behavior configurable (e.g., `forceUppercase: true`).

5. **Error feedback**  
   On API failure the code toasts `resp.error`, which may be undefined.  
   **Fix:** Fallback to a generic message if `resp.error` is missing.

6. **Focus & accessibility**  
   There’s no initial focus on `.x_code`, no focus trap, and close isn’t bound to Escape.  
   **Improvement:** Focus the input on show; optionally bind ESC to close; ensure ARIA roles.

7. **Template & class dependencies**  
   The picker depends on `template: 'codepicker_page'` emitting specific classes (`.bg`, `.pane`, `.x_code`, `.x_confirm`, `.x_close`). Missing classes will break animations/handlers.

8. **Global keyboard dependency**  
   Calls `modules.keyboard_global.hide()` if present; ensure this singleton exists (or guard the call).

9. **Animation timing constants**  
   Durations are hardcoded (`.3s`, `100ms` delay). Consider centralizing or making configurable if you need coherent motion across the app.

10. **Chaining multiple pickers**  
    There’s no queueing/overlay management like the alert plugin. Creating multiple instances in quick succession may stack UIs. Add guards if needed.