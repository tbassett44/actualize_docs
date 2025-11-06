---
title: Phonenumber
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
# Module: `modules.phonenumber` (live phone‐number formatter & validator)

## High-level overview

A tiny UI helper that formats and validates phone numbers **as the user types** by delegating to `libphonenumber`’s parser and “as-you-type” formatter. It keeps the caret sane around edits, emits callbacks when the value becomes valid/invalid, and can prefill from an existing number.  
Reference: **[Google’s libphonenumber](https://github.com/google/libphonenumber)** (parsing/validation semantics).

***

## API surface

### Constructor

`new modules.phonenumber(options)`

| Option       | Type                 | Required | Description                                                              |
| ------------ | -------------------- | -------: | ------------------------------------------------------------------------ |
| `ele`        | jQuery input         |  **Yes** | The `<input>` to bind. Listens to `keydown`, `input paste keyup`.        |
| `onValid`    | `() => void`         |       No | Fired when the current value parses and `isValid() === true`.            |
| `onNotValid` | `() => void`         |       No | Fired when the current value is missing/invalid (see note on bug below). |
| `phone`      | `{ number: string }` |       No | If present, pre-fills the input and triggers a keyup to format/validate. |

> Country is currently hard-coded to **`'US'`** in both the formatter and parser.

### Instance methods

- `getNumber(): PhoneNumber`  
  Parses `self.current` with `libphonenumber.parsePhoneNumber('US')` and returns the parsed object (throws if unparsable—see notes).

- `isValid(number?: string): boolean`  
  Returns `libphonenumber.parsePhoneNumber(x,'US').isValid()` for either the given `number` or the current input value.

- `setCaretPosition(caretPos: number): void`  
  Moves the caret to `caretPos` using `setSelectionRange` (or the legacy `createTextRange` path).

_(`init()` is internal and called automatically by the constructor.)_

***

## How it behaves (under the hood)

1. **Keydown**

   - If **Backspace** (`e.which === 8`), stores `setPosition = selectionStart - 1`.
   - For other keys, resets `setPosition = false`.

2. **Input / Paste / Keyup**

   - Reads the raw value, then formats with `new libphonenumber.AsYouType('US').input(value)`.
   - Writes the formatted string back into the input.
   - If a caret position was staged, calls `setCaretPosition(setPosition)`.
   - Validates via `libphonenumber.parsePhoneNumber(current,'US').isValid()`.

     - On **valid** → calls `options.onValid()` (if provided).
     - On **invalid/parse error** → calls `options.onNotValid()` (intended—see bug).

3. **Prefill**

   - If `options.phone?.number` is set, the constructor sets the input value and triggers `keyup` to format & validate immediately.

***

## Usage

```js
const phoneField = $('#phone');

const ctl = new modules.phonenumber({
  ele: phoneField,
  onValid()   { $('#phone-help').text('✓ Looks good'); },
  onNotValid(){ $('#phone-help').text('Enter a valid US number'); },
  phone: { number: '(303) 555-0107' } // optional prefill
});

// Check validity later (e.g., on submit)
if (!ctl.isValid()) {
  // block submit
}
```

***

## Notes, caveats, and places to harden (poking holes)

1. ### ❗ Callback wiring bug

   In the invalid branches the code checks `if (options.onValid) options.onNotValid()`.  
   If you only supply `onNotValid` (and not `onValid`), it **won’t fire**.  
   **Fix:** change those two lines to `if (options.onNotValid) options.onNotValid();`.

2. ### Country is fixed to `'US'`

   Both the “as-you-type” formatter and the parser use `'US'`. If you need international numbers:

   - Add `options.country` (ISO-2 like `'GB'`, `'FR'`) and use it instead of `'US'`.
   - Consider a separate country selector and re-instantiate or update the country when it changes.

3. ### Caret positioning edge case

   - When Backspace at the **first position**, `selectionStart - 1` is `-1` (or `0` can be falsy). The subsequent `if (self.setPosition)` check will **skip** resetting the caret.  
     **Fix:** check `self.setPosition !== false` and clamp: `Math.max(0, self.setPosition)`.

4. ### `getNumber()` can throw

   `libphonenumber.parsePhoneNumber(...)` throws on some malformed inputs. Wrap it with `try/catch` (the `input` handler already does this).

5. ### Paste & non-digit characters

   The formatter handles pastes well, but if you need stricter control (e.g., strip letters/emojis before formatting), add a pre-sanitization step.

6. ### Library namespace

   The code expects a global `libphonenumber` with `AsYouType` and `parsePhoneNumber`. Ensure you’ve loaded the correct bundle (many projects use `libphonenumber-js` UMD builds).

7. ### Accessibility & UX

   Consider:

   - Setting `inputmode="tel"` on the input.
   - Providing region cues (flag/country code) if you add multi-country support.

***

## Suggested small patch

```diff
 options.ele.on('input paste keyup', function (e) {
   self.current = $(this).val();
   const formatted = new libphonenumber.AsYouType('US').input(self.current);
   $(this).val(formatted);
-  if (self.setPosition) self.setCaretPosition(self.setPosition);
+  if (self.setPosition !== false) self.setCaretPosition(Math.max(0, self.setPosition));

   try {
     const phoneNumber = libphonenumber.parsePhoneNumber(self.current, 'US');
     if (phoneNumber.isValid()) {
       if (options.onValid) options.onValid();
     } else {
-      if (options.onValid) options.onNotValid();
+      if (options.onNotValid) options.onNotValid();
     }
   } catch (e) {
-    if (options.onValid) options.onNotValid();
+    if (options.onNotValid) options.onNotValid();
   }
 });

 this.getNumber = function () {
-  return libphonenumber.parsePhoneNumber(self.current, 'US');
+  try { return libphonenumber.parsePhoneNumber(self.current, 'US'); }
+  catch { return null; }
 };
```

Optionally add `options.country = 'US'` and replace hard-coded `'US'` with `options.country`.

***

If you’d like, I can produce a drop-in **international** version that supports a country dropdown, auto-detects on paste, and stores E.164 (`+1…`) alongside the display format.