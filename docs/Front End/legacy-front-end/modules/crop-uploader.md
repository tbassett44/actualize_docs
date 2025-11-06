---
title: Crop Uploader
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
# Module: `modules.cropuploader` (multi-step image crop + upload flow)

## High-Level Summary

`modules.cropuploader(options)` presents a modal “crop & upload” workflow. Users choose an image, step through **one or more crop presets** (each with its own aspect ratio/constraints), preview the crop region interactively (via **Cropper.js**), and then either (a) upload immediately, or (b) first generate an in-memory cropped preview and then upload. The view slides in with GSAP/TweenLite, supports **Back / Next / Save** navigation, progress bar, background scroll lock, and emits success/progress/exit hooks so you can integrate it with your app and server.

***

## Options / Props

> All options are optional unless marked **Yes**. Some options are passed through to the internal `modules.imageuploader`.

| Name                 | Type                         | Required | Default | Description                                                                                                 |
| -------------------- | ---------------------------- | -------: | ------- | ----------------------------------------------------------------------------------------------------------- |
| `btn`                | `jQuery`                     |       No | —       | Single trigger element. Binds a tap to open the cropper.                                                    |
| `btns`               | `jQuery[]`                   |       No | —       | Multiple trigger elements. Each opens the cropper.                                                          |
| `onClick`            | `() => void`                 |       No | —       | Called when any trigger is tapped before showing UI.                                                        |
| `onBeforeShow`       | `() => void`                 |       No | —       | Called right before the modal is shown.                                                                     |
| `onExit`             | `() => void`                 |       No | —       | Called when the view is destroyed/closed.                                                                   |
| `onCropReady`        | `(imageUrl: string) => void` |       No | —       | Called with a **Blob URL** of the cropped canvas when using `uploadInBackground` (see below).               |
| `onSuccess`          | `(img, instance) => void`    |       No | —       | Called after **successful upload**. `img` has `{ path, ext, ar, v }`.                                       |
| `onProgress`         | `(percent: number) => void`  |       No | —       | Upload progress callback (0–100).                                                                           |
| `disableBodyScroll`  | `boolean`                    |       No | `false` | When true, toggles a `preventScroll` class on `html,body` and blocks touch scroll during the modal.         |
| `uploadInBackground` | `boolean`                    |       No | `false` | If true, begins upload when user taps **Upload** while keeping crop UI light; triggers `onCropReady` early. |
| `exts`               | `string[] \\| false`         |       No | `false` | Allowed file extensions for the underlying uploader.                                                        |
| `directUpload`       | `0 \\| 1`                    |       No | `0`     | Pass-through to `modules.imageuploader` for direct S3 (or similar).                                         |
| `sizes`              | `any`                        |       No | —       | Pass-through to uploader (e.g., server-side resize directives).                                             |
| `returnCropKey`      | `string`                     |       No | —       | If set and a saved crop exists for that key, applies it before extracting the canvas (see **Crops**).       |
| `data`               | **See below**                |  **Yes** | —       | Core configuration for the crop steps and template data.                                                    |

### `data` shape (core)

```ts
data: {
  // Array of crop steps (order = step order)
  crops: Array<{
    width: number;           // target width (used to compute aspect ratio)
    height: number;          // target height (used to compute aspect ratio)
    cropKey: string;         // key used to cache/restore crop box data across steps
    title?: string;          // per-step title (shown in UI)
    responsiveCrop?: boolean // if true, don't force aspect ratio (freeform)
  }>;
  // ... any other values used by your templates
}
```

***

## Events / UI Elements (template contracts)

The templates must render these hooks/classes:

* **Containers & visuals**

  * `.croparea` (crop area wrapper; hidden until an image is selected)
  * `.cropimg` (the `<img>` element passed to Cropper.js)
  * `.bg` (dimmed background)
  * `.pane` (sliding panel container)
  * `.pagesubtitle` (per-step label)

* **Controls**

  * `.x_upload` (upload input/trigger area for `modules.imageuploader`)
  * `.x_save` (**Save/Finish** button)
  * `.x_next` (**Next** step button)
  * `.x_back` (**Back** button)
  * `.x_cancel` (**Cancel/close** button)

* **Bars**

  * `.changebar` (top control bar; shown when an image is loaded)
  * `.stepbuttons` (back/next controls)
  * `.finishbuttons` (final save control)
  * `.progbar` (progress bar wrapper) + `.prog` (inner progress)

> The module attaches handlers to these classes during `binding`.

***

## Dependencies & Side Effects

* **Libraries/Globals**

  * **Cropper.js** (constructor `new Cropper(img, opts)`; used for interactive cropping)
  * **GSAP TweenLite** (`set` / `to` for animation)
  * jQuery
  * Tap abstraction `.stap(...)` (custom)
  * `modules.imageuploader` (internal uploader used under the hood)
  * `modules.present` (modal/presenter manager)
  * `modules.toast(...)` (error toasts)
  * `phi.stop(e)` (stop propagation helper)
  * Env flags: `isPhoneGap()`, `isMobile`
  * App globals: `app.uploadurl`

* **CSS/Body side effects**

  * Adds/removes `preventScroll` class on `html,body` when `disableBodyScroll` is true.
  * Attempts to bind/unbind touch scroll blockade (`ontouchend`—see **Notes**).

***

## How It Works (Flow)

1. **Trigger**\
   Tap `btn`/`btns` → call `onClick?` → `show()` → constructs a `modules.present` view using templates:

   * `templates.alert = 'cropuploader'`
   * `templates.page  = 'cropuploader_mobile'`
   * sets `availWidth` and aspect ratio hints for layout

2. **Bind phase**\
   Inside `binding(ele)`: stores `self.ele`, calls `bind()` to:

   * Initialize an internal `modules.imageuploader` on `.x_upload`
   * Wire all buttons: **Cancel/Back/Next/Save**
   * Wire uploader callbacks (preview, start, progress, success, error)

3. **Preview**\
   On `onPreviewReady(data)`: shows the crop area (hides initial upload prompt), creates or updates a **Cropper** instance, sets the appropriate **aspect ratio** (unless `responsiveCrop`), and caches crop box data per `cropKey`.

4. **Multi-step crop**\
   **Back/Next** moves `cIndex` through `data.crops`. Each step restores any cached crop box for that `cropKey`.

5. **Save/Upload**

   * If **not** `uploadInBackground`: UI shows progress, hides nav; starts upload via `modules.imageuploader.processUpload(self.cropdata)`.
   * If `uploadInBackground`: calls `getCroppedPicture()` first (emits `onCropReady(blobUrl)`), then proceeds to upload.

6. **Finish**\
   On success (`onSuccess`), emits `{ path, ext, ar, v }` and keeps the instance for the caller. On error, shows a toast and resets progress state.

***

## Option Details

### Aspect Ratio logic

* For the current step `k = crops[cIndex]`:

  * If `k.responsiveCrop` is **true**, aspect ratio is **not** enforced (returns `false` to Cropper).
  * Else aspect ratio = `k.width / k.height`.
* Crop box data is stored/restored in `self.cropdata[k.cropKey]` via `crop` callback and `setData()`.

### Buttons & State guards

* **Next/Back**: navigates steps; **Save** on final step calls `processUpload()`.
* **isSaving**: prevents double submit; `saving()` swaps `.x_save` label to a spinner and back on error.
* **ensureButtons()**: toggles visibility for `stepbuttons`, `finishbuttons`, `changebar`, and hides **Back** at the first step.

### Upload integration (`modules.imageuploader`)

* Initialized with:

  * `ele: .x_upload`
  * `apiurl: app.uploadurl`
  * `exts, directUpload, data: { sizes, path:'/upload/' }` (path static here)
* Hooks:

  * `onPreviewReady(data)`: provides a **data URL** to initialize Cropper (sets image src)
  * `onUploadStart()`: toggles bars and optionally triggers `getCroppedPicture()` if `uploadInBackground`
  * `onProgress(p)`: updates `.prog` width and calls `options.onProgress`
  * `onSuccess(obj, resp)`: builds `{ path, ext, ar, v }` and calls `options.onSuccess(img, self)`
  * `onError(msg)`: toasts the error and resets UI

***

## Usage Example

```js
new modules.cropuploader({
  btn: $('#changeAvatar'),
  disableBodyScroll: true,
  uploadInBackground: false,
  exts: ['jpg','jpeg','png','webp'],
  sizes: { avatar: { w: 512, h: 512, fit: 'cover' } },
  data: {
    crops: [
      { width: 1, height: 1, cropKey: 'avatar_square', title: 'Adjust your avatar' }
    ]
  },
  onBeforeShow() {
    console.log('Opening cropper…');
  },
  onProgress(p) {
    // update a custom UI too, if desired
  },
  onSuccess(img /* {path, ext, ar, v} */, instance) {
    // e.g., update UI with new URL + cache bust
    const url = `${img.path}?v=${img.v}`;
    $('.profile .avatar').attr('src', url);
    instance.destroy();
  },
  onExit() {
    console.log('Cropper closed');
  }
});
```

**Multiple steps example** (banner + thumbnail):

```js
new modules.cropuploader({
  btn: $('#editPhotos'),
  data: {
    crops: [
      { width: 1200, height: 400, cropKey: 'banner',  title: 'Banner (3:1)' },
      { width:  400, height: 400, cropKey: 'thumb',   title: 'Thumbnail (1:1)' }
    ]
  },
  onSuccess(img) {
    console.log('Uploaded!', img);
  }
});
```

***

## Notes & Edge Cases (rigorous checks)

1. **Template contracts are required**\
   Missing expected classes (`.x_upload`, `.cropimg`, `.x_save`, etc.) will break bindings. Ensure your `cropuploader` / `cropuploader_mobile` templates match.

2. **Touch scroll blocker event name**\
   The code uses `$('html,body').on('ontouchend', self.onScroll)` and `.off('ontouchend', ...)`. In jQuery the event is **`'touchend'`not`'ontouchend'`** .\
   **Recommendation:** change to `'touchmove'`/`'touchstart'`/`'touchend'` as appropriate.

3. **Aspect ratio flag**\
   `responsiveCrop` disables aspect ratio enforcement. If omitted or falsy, `width/height` must be valid numbers; otherwise `NaN` AR will break Cropper.

4. **State resets**

   * `clearUploadProcess()` is called only on `onError`; ensure any **manual aborts** (`.x_cancel`) also reset UI if needed.
   * `destroy()` cleans cropper, removes scroll lock, and calls `onExit()`.

5. **Background upload +`onCropReady`**\
   When `uploadInBackground` is true, `getCroppedPicture()` produces a **Blob URL** from the current crop and calls `onCropReady(imageUrl)`. Revoke it when done: `URL.revokeObjectURL(url)`.

6. **Progress UI**\
   `.progbar` and `.prog` must exist. Without them, progress updates are no-ops on the DOM.

7. **Multiple steps & cached data**\
   Crop data is cached per `cropKey`. If you pass `returnCropKey`, `getCroppedPicture()` will apply that cached data **before** extracting the blob, which is useful if you want to ensure a specific step’s crop is used.

8. **Uploader path/data**\
   The uploader posts with `data: { sizes, path: '/upload/' }`. If your backend expects a different path or additional fields, pass them through (or adapt `modules.imageuploader`).

9. **Memory & canvas limits**\
   Very large images can cause canvas memory issues when calling `getCroppedCanvas().toBlob(...)`. Consider constraining maximum output size in Cropper options or server-side.

10. **Accessibility**\
    There’s no built-in focus management or keyboard bindings (ESC to close, arrow keys to nudge). Consider adding for a11y.

***

## Quick Patches (optional)

* **Fix scroll blocker event names**

  ```js
  // Instead of 'ontouchend'
  $('html,body').on('touchmove', self.onScroll);
  // and later
  $('html,body').off('touchmove', self.onScroll);
  ```

* **Guard aspect ratio math**

  ```js
  this.getAr = function (cropper) {
    const d = options.data.crops[self.cIndex];
    if (cropper && d.responsiveCrop) return false;
    const ar = Number(d.width) / Number(d.height);
    return Number.isFinite(ar) ? ar : false;
  };
  ```

* **Revoke Blob URLs after use**\
  If you keep the preview around, remember to `URL.revokeObjectURL(imageUrl)` when it’s no longer needed.

If you’d like, I can produce a **type-annotated** (TS JSDoc) version of this doc or a **refactor patch** that addresses the event name, AR guards, and optional a11y bindings.
