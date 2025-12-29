---
title: File Uploading
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
# Module: `modules.fileuploader` (general file upload with preview/progress)

## High-Level Summary

`modules.fileuploader(opts)` wraps a third-party uploader (`ss.SimpleUpload`) to provide file selection, client-side preview, progress UI, size/type validation, and completion/error hooks. It can upload either via the SimpleUpload flow **or** via a bespoke `submitBlob` (FormData/XHR) path. It also integrates with app-level UX utilities (`modules.toast`, `ele.spin`), and supports PhoneGap uploads via `app.core.camera.uploadimg`.

***

## Options / Props

> All options are optional unless marked **Yes**. Types inferred from usage.

| Name                | Type                                      | Required | Default       | Description                                                                                                                                                                        |
| ------------------- | ----------------------------------------- | -------: | ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ele`               | `jQuery`                                  |  **Yes** | —             | The clickable element that acts as the upload button (its DOM node is passed to SimpleUpload as `button`).                                                                         |
| `module`            | `string`                                  |       No | `'file'`      | Used to build upload URLs: `/upload/{module}/submit` and `/upload/{module}/progress`.                                                                                              |
| `allowedExtensions` | `string[]`                                |       No | `['pdf']`     | Valid file extensions (case-insensitive; also used to gate preview generation).                                                                                                    |
| `multiple`          | `boolean`                                 |       No | `false`       | Allow selecting multiple files (`1` passed to uploader).                                                                                                                           |
| `previewOnly`       | `boolean`                                 |       No | `false`       | If true, uploader configured to preview without immediate upload (passed through as `1`).                                                                                          |
| `maxSize`           | `number`                                  |       No | `5000000`     | Labeled “kilobytes” in code comment, but numeric value suggests **bytes** (see Notes).                                                                                             |
| `nospin`            | `boolean`                                 |       No | `false`       | If falsey, `ele.spin({size:16})` is shown during upload and cleared later.                                                                                                         |
| `onSubmit`          | `(fileMeta) => void`                      |       No | —             | Called when a file is selected and submission is about to start. Receives `{ size, ext, file, mbsize }` via uploader `obj`. If not provided, a default toast+progress UI is shown. |
| `onLoadingPreview`  | `() => void`                              |       No | —             | Called before generating local preview.                                                                                                                                            |
| `onPreviewReady`    | `(localUrl, fileMeta) => void`            |       No | —             | Provides a blob/data URL for client-side preview (image/video/etc.).                                                                                                               |
| `onAbort`           | `() => void`                              |       No | —             | Called when an upload is aborted.                                                                                                                                                  |
| `onError`           | `(msg, obj, b?, c?) => void`              |       No | default toast | Error handler; default shows a warning toast and stops spinner.                                                                                                                    |
| `onDone`            | `(fileMeta, response) => void`            |       No | default toast | Success handler when the SimpleUpload completes. Aliased from `onComplete` if you pass that instead.                                                                               |
| `onUploadStart`     | `() => void`                              |       No | —             | Called at the start of **manual** upload via `processUpload` or PhoneGap path.                                                                                                     |
| `onSuccess`         | `(fileMeta, resp, originalFile?) => void` |       No | —             | Called after **PhoneGap** upload success (`processUpload` mobile branch).                                                                                                          |

> **URL base:** `app.uploadurl` is expected globally.\
> **Auth:** `app.user.token` and `app.appid` are injected before uploading (web branch).

***

## Methods (instance)

| Method          | Signature                                      | Returns                 | Description                                                                                                                                                                                                                                         |                                                                                                                                                                          |
| --------------- | ---------------------------------------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `isUploading`   | `() => boolean`                                | `true/false`            | Current upload flag.                                                                                                                                                                                                                                |                                                                                                                                                                          |
| `hasVideo`      | `() => boolean`                                | `true/false`            | Indicates a file has been queued/loaded (set in `onQueue`).                                                                                                                                                                                         |                                                                                                                                                                          |
| `abort`         | `(id: string) => void`                         | `void`                  | Triggers click on `#${id}_abort` (requires corresponding abort control in DOM—see Notes).                                                                                                                                                           |                                                                                                                                                                          |
| `destroy`       | `() => void`                                   | `void`                  | Destroys the underlying `SimpleUpload` instance.                                                                                                                                                                                                    |                                                                                                                                                                          |
| `processUpload` | `(data: object) => void`                       | `void`                  | **Manual upload trigger**. - **PhoneGap:** uses `app.core.camera.uploadimg`; calls `onUploadStart`, then `onSuccess`/`onError`. - **Web:** merges `data` into `uploader._opts.data`, injects `appid`/`token`, and calls `uploader.processUpload()`. |                                                                                                                                                                          |
| `submitBlob`    | \`(blob: Blob, form\_data?: object, cb?: (file | false)=>void) => void\` | `void`                                                                                                                                                                                                                                              | Sends a **FormData** POST to `/upload/{module}/submit`. Calls `cb({ path, meta })` on success, or `cb(false)` and toasts on error. Simulates progress via `setInterval`. |
| `localUrl`      | `(file: File, cb: (url:string)=>void) => void` | `void`                  | Produces a preview URL via `URL.createObjectURL(file)` (or `FileReader` data URL fallback) and calls `cb(url)`.                                                                                                                                     |                                                                                                                                                                          |

***

## Uploader Configuration (internal defaults)

```js
{
  button: ele[0],
  url:        app.uploadurl + '/upload/' + (opts.module || 'file') + '/submit',
  responseType: 'jsonp',
  progressUrl: app.uploadurl + '/upload/' + (opts.module || 'file') + '/progress',
  name: 'file',
  cors: true,
  multiple: opts.multiple ? 1 : 0,
  previewOnly: opts.previewOnly ? 1 : 0,
  allowedExtensions: extensions,
  hoverClass: 'hover',
  maxSize: 5000000,
  onSubmit(obj) { ... },
  onQueue(obj) { ... },
  onSizeError(obj) { ... },
  onExtError(obj) { ... },
  onAbort() { ... },
  onComplete(obj, response) { ... }
}
```

* **Progress UI:** If you don’t provide `onSubmit`, a toast with template `'fileupload'` is shown and its `.progbar` is bound via `this.setProgressBar(...)`; file size displayed in `.uploadsize`.
* **Preview:** In `onQueue`, if `noPreview` is not set, `localUrl` is used to produce a preview for files whose extension is in `allowedExtensions`. (Note: current code **does not** error on unsupported types; it simply skips preview.)

***

## Dependencies & Side Effects

* **Libraries/Globals:**

  * `ss.SimpleUpload` (third-party uploader)
  * `jQuery`
  * `modules.toast(...)` (UI toasts)
  * `ele.spin(...)` (spinner plugin)
  * `Math.uuid(...)`
  * `app.uploadurl`, `app.user.token`, `app.appid`
  * PhoneGap branch: `isPhoneGap()`, `app.core.camera.uploadimg`
* **DOM expectations (default UI path):**

  * A toast template `'fileupload'` that includes `.progbar` and `.uploadsize`.
* **Network:**

  * Default `responseType: 'jsonp'` and `progressUrl` (server must support JSONP progress polling).
* **Auth injection (web branch):**

  * Writes into **private** field `self.uploader._opts` before upload: `data`, `token`, `appid`.

***

## Usage Examples

```js
// Basic: PDF uploader with preview + progress toast
const fu = new modules.fileuploader({
  ele: $('#uploadBtn'),
  allowedExtensions: ['pdf'],
  onPreviewReady(url, obj) {
    $('#previewFrame').attr('src', url);
  },
  onDone(fileMeta, resp) {
    modules.toast({ content: 'Upload complete!', remove: 2000 });
  },
  onError(msg) {
    modules.toast({ content: 'Upload failed: ' + msg, type: 'warning' });
  }
});
```

```js
// Image uploader with manual process + extra form fields
const imgUp = new modules.fileuploader({
  ele: $('#uploadImage'),
  module: 'image',
  allowedExtensions: ['jpg','jpeg','png','webp'],
  onSubmit(file) {
    // show your own UI instead of default toast
    $('#status').text(`Uploading ${file.mbsize} MB…`);
  },
  onDone(file, resp) {
    $('#status').text('Done!');
    $('#img').attr('src', resp.path + '?v=' + (resp.v || 0));
  }
});

// Later, when you’re ready to push the upload with extra data (e.g., crop info)
imgUp.processUpload({ sizes: { cover: { w: 1200, h: 600 } }, path: '/upload/' });
```

```js
// Direct FormData upload (e.g., microphone blob)
const blob = await getAudioBlobSomehow();
fu.submitBlob(blob, { path: '/audio/', site: 'nectar' }, (file) => {
  if (!file) return modules.toast({ content: 'Audio upload failed' });
  console.log('Uploaded audio to', file.path, 'meta:', file.meta);
});
```

***

## Notes & Edge Cases (rigorous checks)

1. **Size units mismatch**\
   `maxSize: 5000000` is labeled “kilobytes” in a comment, but the value looks like **bytes (\~5 MB)**. Verify `ss.SimpleUpload` expects **bytes**; if it expects **KB**, this is \~5 GB (!) and size checks won’t work.

2. **Preview type gating**\
   `onQueue` checks `extensions.indexOf(obj.ext) >= 0` before preview. For non-whitelisted types, no preview, no error. If you want a clear message, call `opts.onError` there.

3. **Abort mechanism**\
   `abort(id)` triggers `#${id}_abort`. This assumes an element exists with that id; the code that created such a button is commented out (`setAbortBtn`). As-is, `abort()` likely does nothing. Either wire `setAbortBtn` or expose `self.uploader.abort()` if available.

4. **Private field mutation**\
   Writing to `self.uploader._opts` is **internal API** of SimpleUpload; library updates might break this. Prefer public setters if the lib offers them.

5. **Auth availability**\
   If `app.user.token` is absent, uploads will include an undefined token. Consider guarding and surfacing a clear error.

6. **`responseType: 'jsonp'`+`progressUrl`**\
   Your server must support **JSONP** for completion and progress polling; otherwise use CORS JSON/XHR. JSONP also implies GET semantics—ensure that matches server behavior.

7. **`localUrl`implementation quirk**\
   The function redeclares `reader` and then assigns **`window.URL || window.webKitURL`** to it; `webkitURL` should be lowercase (`window.webkitURL`). Also, the variable `url` is used without `var/let/const`. Consider fixing for reliability and strict mode.

8. **Mobile / PhoneGap branch**\
   The PhoneGap path goes through `app.core.camera.uploadimg(self.data, qsdata, self.fileobj, cb)`. Ensure these fields are populated (`self.data`, `self.fileobj`). Otherwise you’ll see failures. This branch calls `onSuccess(fileobj, resp, fileobj)` on success.

9. **Spinner & toasts coupling**\
   Default `onError` stops spinner for you **unless** `nospin` is true. If you override `onSubmit` with custom UI, you must handle spinners yourself.

10. **Security**\
    Never trust extension checks alone—server must validate MIME and sanitize filenames. Consider adding content-type checks in client if helpful.

***

## Quick Patches (optional)

* **Fix`localUrl` robustness**

  ```js
  this.localUrl = function (file, cb) {
    const URL_ = window.URL || window.webkitURL;
    if (URL_ && URL_.createObjectURL) {
      const url = URL_.createObjectURL(file);
      cb(url);
      // URL_.revokeObjectURL(url); // revoke when you’re done showing preview
    } else {
      const fr = new FileReader();
      fr.onload = (e) => cb(e.target.result);
      fr.readAsDataURL(file);
    }
  };
  ```

* **Clarify size units**\
  Decide on bytes vs KB and update both `maxSize` value and comment. For **5 MB** with bytes: `maxSize: 5 * 1024 * 1024`.

* **Expose a public abort** (if SimpleUpload supports it)

  ```js
  this.abort = function () {
    if (self.uploader && self.uploader.abort) self.uploader.abort();
  };
  ```

* **Avoid`_opts` mutation**\
  If possible, construct a fresh `SimpleUpload` with desired `data`, `appid`, `token` or use documented setters.
