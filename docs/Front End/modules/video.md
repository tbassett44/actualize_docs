---
title: Video
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
# Module: `modules.video` — Video playback (Plyr), viewing, capture & upload

## High-level overview

This module wraps **[Plyr](https://github.com/sampotts/plyr)** for consistent HTML5/YT playback across devices, plus a full flow for **picking/capturing** videos on mobile (Cordova), **uploading** (foreground or managed background), optional **transcoding**, progress UIs, and a lightweight **viewer** overlay. It also coordinates orientation and status-bar behavior, and ensures only one video plays at a time. 

***

## Public surface (modules & roles)

* **`modules.video_player(options)`** – Creates a Plyr instance on a `<video>` element with sensible defaults for web vs. mobile, iOS native fullscreen, inline playback, and event hooks (time tracking, fullscreen, ready, play/pause).
* **`modules.video_view(media)`** – Full-screen overlay “viewer”; renders a template, instantiates an internal `modules.video_player`, auto-plays, and wires a close button that restores orientation/status bar.
* **`modules.video(options)`** – End-to-end **acquire + upload** controller. Handles web file uploads, PhoneGap media picking/capture, optional background uploads, and (when available) client-side transcoding before upload. Exposes onPreview/onProgress/onSuccess hooks.
* **`modules.video_preview`** – Small helpers to render a preview template and repair iOS status bar on exiting native fullscreen.
* **`modules.video_global`** – Global state: `current` (the active player) and `isUploading` flag.

***

## `modules.video_player(options)`

### Options

| Option         | Type                                | Default  | Purpose                                                                                                                                             |
| -------------- | ----------------------------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ele`          | jQuery                              | —        | The `<video>` element to control.                                                                                                                   |
| `opts`         | object                              | built-in | Passed to Plyr: control set varies for web vs. app; `settings: ['quality','speed']`; `playsinline: true`; iOS native fullscreen toggled by default. |
| `embed`        | boolean                             | `false`  | If true, disables Plyr fullscreen (or delegates to native YT player on dev iOS if `youtube_id` provided).                                           |
| `youtube_id`   | string                              | —        | Used with `embed` to open the native YouTube player (dev iOS path).                                                                                 |
| `onTimeUpdate` | fn(\{duration,currenTime,playTime}) | —        | Called with total duration, current time, and **cumulative watch time** (resilient to small seeks).                                                 |

### Behavior & events

* Builds a Plyr instance and **prevents swipe-back** conflicts while scrubbing (`touchmove`/`touchend` set `phi.preventSwipe`).
* iOS: binds `webkitendfullscreen` to ensure status bar is restored when native fullscreen ends.
* **Time tracking**: accumulates `playTime` provided small jumps `<2s`.
* **Fullscreen**: on enter → unlock orientation and hide status bar (after a short delay); on exit → lock orientation and show status bar.
* **Single-play policy**: when a player starts, it pauses any previously playing instance via `modules.video_global.current`.

### Methods

* `play()`, `pause()`, `autoplay()` (mutes first if needed), `destroy()`.

***

## `modules.video_view(media)`

One-call overlay viewer:

* Renders the `video_view` template with `media`, unlocks orientation, hides status bar, instantiates `modules.video_player`, auto-plays, and shows a dim background.
* Close (`.x_close`) locks orientation, shows status bar, pauses, destroys the player, and removes the overlay.

***

## `modules.video(options)` – capture/pick & upload

### Typical web flow

* Internally creates a `modules.fileuploader` bound to `options.ele`.
* Accepts only `mp4` (others can be enabled once transcoding pipeline is ready).
* Emits:

  * `onUploadStart()` when the file is submitted
  * `onPreviewReady(localUrl, fileObj)` for a local preview
  * `onProgress(percent)` during upload
  * `onSuccess(file, response)` on completion
  * `onError(fileObj, err)` on failure

### Typical mobile (Cordova) flow

* Tapping `options.ele` opens a small **mobilealert** menu (e.g., “Upload a video”).
* **Pick from gallery** via `modules.mediapicker` (preferred) or, in some paths, `navigator.camera.getPicture` with `mediaType: VIDEO`.
* **(Optional) Background upload** (iOS with `FileTransferManager`): queue via `phone.bg_uploader`; otherwise use standard `FileTransfer`.
* **(Optional) Transcoding** (when `window.VideoEditor` present and `options.transcode` true): transcodes first, then uploads.
* Shows a **progress tray** (`video_upload_tray` / `video_progress` templates) with cancel support; also supports a **growl** upload status in web layout.

### Key options & hooks

| Option                               | Type    | Purpose                                                                                          |
| ------------------------------------ | ------- | ------------------------------------------------------------------------------------------------ |
| `ele`                                | jQuery  | Trigger element to bind (click/tap).                                                             |
| `immediateUpload`                    | boolean | If true, begins upload immediately after selecting the video.                                    |
| `transcode`                          | boolean | If true and the plugin exists, transcode before upload (bitrate, fps, etc., are set internally). |
| `data`                               | object  | Extra form data to include with uploads (merged with `appid`/`token`).                           |
| `onClick`                            | fn      | Called when the trigger element is tapped/clicked.                                               |
| `onPreviewReady(pathOrUrl, fileObj)` | fn      | Local preview (file path on mobile, blob URL on web).                                            |
| `onUploadStart()`                    | fn      | Called when upload starts.                                                                       |
| `onProgress(p)`                      | fn      | Upload progress (0–100).                                                                         |
| `onSuccess(obj, resp)`               | fn      | Called when server responds OK.                                                                  |
| `onError(fileObj?, err?)`            | fn      | Called on failure/abort.                                                                         |

### Useful instance methods

* `process(obj?)` – Switches to “upload in progress” UI, then performs (transcode →) upload.
* `upload_background(path, opts, fileObj, cb)` – Route to background upload if possible; otherwise falls back to direct upload.
* `upload(path, opts, fileObj, cb)` – Direct Cordova `FileTransfer` upload.
* `abort()` – Cancels active upload (background or foreground).
* `isUploading()` – Returns current upload state.
* `hasVideo()` / `clearVideo()` – Manage selected media path (mobile).
* `checkUpload(opts, cb)` – If uploading, shows a prompt; otherwise runs `cb()`.

**Server contract (success)**

```
{ path, ar, ext, poster, length }
```

Returned by the upload endpoint and re-emitted to your `onSuccess` handler.

***

## Templates this module expects/uses

* `video_preview`, `video_view`, `video_upload_tray`, `video_progress`, `uploader_prompt`, `uploader_prompt_web`, plus your item/card templates that render uploaded media.

***

## Minimal examples

### A) Inline player

```js
const vp = new modules.video_player({
  ele: $('video.my-clip'),
  onTimeUpdate(e) {
    // e.playTime is “seconds actually watched”
  }
});
```

### B) One-tap viewer overlay

```js
$('.open-video').on('click', () => new modules.video_view({
  src: 'https://cdn.example.com/video.mp4',
  poster: 'https://cdn.example.com/poster.jpg'
}));
```

### C) Web upload with preview + progress

```js
new modules.video({
  ele: $('#uploadVideo'),
  immediateUpload: true,
  data: { path: '/video/' },
  onPreviewReady(url) { $('#preview video').attr('src', url); },
  onUploadStart() { $('#progress').show(); },
  onProgress(p) { $('#progress .bar').css('width', p + '%'); },
  onSuccess(obj, resp) { console.log('Uploaded to', resp.path); },
  onError(_, err) { modules.toast({ content: 'Upload failed' }); }
});
```

### D) Mobile pick + background upload (iOS)

```js
new modules.video({
  ele: $('.mobile-upload'),
  immediateUpload: true,
  transcode: true, // if VideoEditor plugin is installed
  onPreviewReady(localPath) { /* show thumbnail */ },
  onSuccess(_, resp) { /* attach resp.path to form */ }
});
```

***

## Considerations & hardening (poke the edges)

* **One-player policy:** The global `modules.video_global.current` approach ensures only one video plays, but if you create/destroy many players, keep `destroy()` tidy to avoid stale references.
* **Fullscreen logic:** iOS native fullscreen + delayed orientation unlock (5s) can feel odd; consider making the delay configurable or listening for real state.
* **Transcoding availability:** Guard UI for transcode-only features; fall back cleanly if the plugin or background uploader isn’t present.
* **MIME & formats:** Direct `FileTransfer` uses `"video/quicktime"`; if you upload MP4 broadly, consider detecting/setting MIME by extension.
* **Error paths:** Surface server errors up to users (quota, file too large, unsupported codec), not just a generic “error”.
* **Security:** Uploads include `appid/token`; ensure server-side validation and size limits.
* **Accessibility:** Provide captions/subtitles (VTT) and expose Plyr’s accessibility features in templates.
