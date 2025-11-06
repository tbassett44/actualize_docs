---
title: QRcode Scanner
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
# Module: `modules.qrcode` (QR code scanning & generation)

**High-level overview:**\
A compact wrapper around the **BitPay Cordova QRScanner plugin** for **scanning** QR codes in Cordova/PhoneGap builds, plus a small helper to **generate** QR images as base64. It handles permission prompts, status-bar styling, background clearing, scan loop management, success/fail UI, and optional server-side validation of scanned tokens.\
Docs: **BitPay QRScanner** → [https://github.com/bitpay/cordova-plugin-qrscanner](https://github.com/bitpay/cordova-plugin-qrscanner). 

***

## Public surface

### `modules.qrcode.scanner.show(opts)`

Start the scanner UI.

* **Environment gate:** if not PhoneGap and not dev (`!isPhoneGap() && !app.isdev`), it shows a “coming soon” message and returns. Otherwise it preps device UI and proceeds.
* **State captured:** remembers current status-bar theme, resets `scanstarted/lighton`, stores `opts` and the container `opts.ele`.
* **Cordova path:** calls `load()` (prepares plugin, then `start()`); **web/dev path:** clears app background to reveal the preview layer. 

**`opts`(commonly used):**

* `ele`: jQuery element that contains scanning UI (used to render success/fail overlays).
* `onShow()`: called when preview is shown.
* `onScan(text)`: called with scanned string.
* `validateUrl`: if provided, each scan is POSTed for validation (see `validate`).
* `templates: { success, fail }`: template names to render transient feedback in `.successarea`.

***

### `modules.qrcode.scanner.hide()`

Stop and tear down the scanner.

* Restores status bar theme, resets background, clears any running scan timeouts, and **destroys** the native scanner via `QRScanner.destroy`. 

***

### `modules.qrcode.scanner.pause()` / `resume()`

Pause/resume the live camera preview (`QRScanner.pausePreview` / `resumePreview`). Idempotent. 

***

### `modules.qrcode.scanner.load()`

Prepare the plugin and kick off scanning.

* Ensures `window.QRScanner` exists; calls `QRScanner.prepare((err,status) => {...})`.
* Stores permission state:

  * `status.authorized` → ready to scan.
  * `status.denied` → permanently denied; suggest `QRScanner.openSettings()` externally.
  * else → temporarily denied; may retry later.
* Clears background and calls `start()`. 

***

### `modules.qrcode.scanner.start(retry?)`

Show camera preview and begin scanning.

* Sets status-bar theme to **light** (for contrast), calls `QRScanner.show(onShown)`, then `scan()`.
* If still “preparing”, retries up to \~2s in 50ms steps. 

***

### `modules.qrcode.scanner.stop()`

Stops the scanning loop (clears the internal `scanTimeout` and marks `scanActive=false`). 

***

### `modules.qrcode.scanner.scan()`

One-shot scan with auto-rearm.

* Calls `QRScanner.scan((err,text) => {...})`.
* On success: triggers `onScan(text)` then **re-arms** the scanner after **2.5s** (`setTimeout(this.scan, 2500)`) to give the camera time to settle.
* On error (including cancel): silently returns. (Your code can call `start()` again if desired.) 

***

### `modules.qrcode.scanner.onScan(text)`

Unified scan handler:

* Invokes `opts.onScan(text)` if present.
* Calls `validate(text, cb)`; on success it renders the **success** template (see below). 

***

### `modules.qrcode.scanner.validate(ticket_id, cb)`

Optional server validation:

* If `opts.validateUrl` is set, performs `app.api` request with `{ ticket: ticket_id }`.

  * On `{ success: true, valid: true }` → `cb(true, resp.scan)`
  * On error/invalid → renders **fail** toast and `cb(false)`
* If no endpoint is configured, **treats as valid** and returns a canned object for display. 

***

### `modules.qrcode.scanner.onSuccess(scan)` / `onError(msg)`

Transient overlays:

* Renders into `opts.ele.find('.successarea')` using `opts.templates.success` or `opts.templates.fail`; each fades out after \~5s. 

***

### `modules.qrcode.scanner.test(id)`

Dev helper: manually feed a code string through `onScan` + `validate` and show success overlay if it passes. Useful on web while building the UI. 

***

### `modules.qrcode.drawGuide(canvas, success?)`

Draws a **semi-transparent dim** with a rounded square “hole” and colored corner marks (red or green when `success` truthy). Auto-sizes to viewport; good for overlaying scan regions. 

***

### Generator helpers

#### `modules.qrcode.ensure(cb)`

Ensures a hidden DOM container (template: `'qrcode'`) exists; passes it to `cb` when ready. 

#### `modules.qrcode.getBase64(content, cb)`

Generates a **QR image** for `content` using a `QRCode` library instance (300×300, high error correction) and returns a **data URL** via `cb`. Reuses the same element across calls. 

***

## Typical usage

```js
// 1) Show scanner
modules.qrcode.scanner.show({
  ele: $('#qrScannerContainer'),
  onShow() { /* bind overlay, vibrate, etc. */ },
  onScan(text) { console.log('Scanned:', text); },
  validateUrl: app.apiurl + '/tickets/validate',
  templates: {
    success: 'qr_success_toast',
    fail:    'qr_fail_toast'
  }
});

// 2) Later: pause/resume or hide
modules.qrcode.scanner.pause();
// ...
modules.qrcode.scanner.resume();
// ...
modules.qrcode.scanner.hide();

// 3) Generate a QR for sharing
modules.qrcode.getBase64('user:12345', (dataUrl) => {
  $('#myQr').attr('src', dataUrl);
});
```

***

## Dependencies & integration points

* **Cordova plugin:** `cordova-plugin-qrscanner` (BitPay) — camera preview behind the webview, scanning, permissions.
* **UI/Device helpers:** `phone.statusBar`, `phone.background` for visual polish while scanning.
* **Templates:** You supply `opts.templates.success` / `fail` (rendered in `.successarea` within `opts.ele`).
* **Networking:** `app.api` for `validateUrl` calls (server decides “valid”/“invalid”).
* **Generator:** global `QRCode` class (e.g., `davidshimjs-qrcode`) to create base64 images. 

***

## Edge cases & hardening (poking holes)

1. **Web support:** Non-Cordova path in production does not scan; it only clears background. Use `scanner.test(id)` for manual testing, or feature-detect and hide the scan entry point on web. 
2. **Permission handling:** When `status.denied`, consider surfacing a UI control to call `QRScanner.openSettings()` so users can re-enable the camera. Current code only sets `self.error='denied'` and alerts. 
3. **Validation transport:** The code uses `app.api` (not `modules.api` like other modules). Ensure `app.api` exists and aligns with your API stack—or switch to your standard wrapper for consistency. 
4. **Scan re-arm delay:** The fixed **2.5s** delay between scans may feel slow for batch scanning. Make it configurable (`opts.scanDelay`) if you need faster throughput. 
5. **Cleanup guarantees:** `hide()` calls `QRScanner.destroy`; ensure it’s also invoked on route changes/unloads to release the camera promptly. 
6. **Status-bar theme:** `start()` unconditionally sets `'light'`. Consider restoring immediately on pause or when overlays are bright to maintain contrast. 
7. **Templates contract:** Success/fail renderers require `.successarea` under `opts.ele` and your named templates to exist; otherwise overlays won’t appear.
