---
title: Image Processing
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
# Image Processing & URL API — `img.actualize.earth`

## Overview

This service wraps **GraphicsMagick (`gm`)** to resize, crop, transform, and optionally grayscale images. It also includes helpers to generate QR codes, base64-encode results, and (internally) batch-upload variants to S3.

- Core class: `ImageResizer`
- Key methods: `render`, `getSizing`, `resizeImage`, `outputToHeaders`
- Extras: QR generation (`generateQR`, `generateQR2`), GIF first-frame extraction, base64 output, S3 multi-size upload helper.

> Note: The file sets strong caching headers (ETag + far-future expiry) on successful responses and will 304 if the client’s ETag matches.

***

## Base URL

```
https://img.actualize.earth
```

## Path format

The service expects the image path in the URL, and reads the **bucket** from the second path segment:

```
/{bucket}/{path/to/image}.{ext}?{querystring}
```

- In code, `bucket` is taken from `explode('/', $source)[1]`.
- Remote sources (`http(s)://…`) are supported: the service will download to a temp cache before processing.

***

## Query String Parameters

### Sizing & Cropping

- `width` — max/output width in pixels (int)
- `height` — max/output height in pixels (int)
- `crop` — if present (any truthy), perform a **crop to fit** using `width`×`height` as the final canvas; centers and crops based on aspect ratio
- `cropratio` — alternative crop target ratio instead of explicit `width`/`height` (e.g., `1:1`, `3:2`)
- `square` — if present, fit the image inside a square (paired with `width` and/or `height`)

### Color / Effects

- `gray` — if present, convert final image to grayscale

### Output / Encoding

- `quality` — JPEG/PNG quality hint passed to GraphicsMagick (int)
- `base64` — if present, returns a `data:image/*;base64,...` string instead of binary image data

### Caching Controls

- `nocache` — bypass reading a previously rendered cached file; forces regeneration
- `uncache` — destroy the cached render and **do not** display (maintenance/flush behavior)

### Advanced Crop Data (power user)

- `cropdata` — structured crop info (the code calculates `cropstring` with width/height/offset).  
  The implementation parses percentage-based size/location and an ending pixel size, then builds a GM `-crop` string with offsets. 

### Notes on query parsing & cache key

- The cache key is derived from the URL’s querystring **minus** some control keys. The code explicitly **excludes** `height`, `width`, `nocache`, and `hex` from the stored query text, and it avoids assigning `rgb` into the object when parsing the raw query tail. (I don’t see first-class `hex`/`rgb` features wired into transforms in this file; they appear in the allow/deny list only.)
- Internal flags encountered in code: `force` (used to avoid a degenerate “no-resize” case when width matches original); not explicitly exposed in the public param list.

***

## Behavior Details

### Sizing logic (`getSizing`)

- Reads source dimensions and mime; validates mime starts with `image/`.
- Computes `outwidth`/`outheight` based on:

  - `crop` + (`width` & `height`): center-crop to match target aspect, then size to exact `width`×`height`.
  - `cropratio`: computes a crop box that fits the ratio, then resizes.
  - Else: fits within `maxwidth`×`maxheight` without cropping (preserves aspect).

### Rendering (`resizeImage`)

- Handles JPEG EXIF orientation (auto-orient via GM flags; legacy EXIF branch is commented).
- Applies:

  - `-size WxH`, `-density 72x72`
  - grayscale if `gray`
  - quality if `quality`
  - crop path if `crop` or advanced `cropdata`
- Writes to a cache file in `ROOT./imagecache/` (name is md5 of cache text + image name).
- ETag is md5 of the output data; returns 304 when `If-None-Match` matches.

### GIF first frame

- `getGifFrame(url)` downloads the GIF, then runs:

  ```
  gm convert {gif}[0] {tmp}_frame.jpeg
  ```

  and uses that frame for processing.

### Remote sources

- `save($path, $saveto = false, $nocache = false)` downloads remote images (with a desktop-like header set) into `/tmp/imagecache/{md5(url)}` and reuses them unless `nocache`.

***

## Response

- **Default**: binary image with `Content-Type` set to the detected mime; cache headers + ETag.
- **When `base64` is present**: returns a `data:image/{jpeg|png};base64,...` string.
- On errors: `400 Bad Request` for invalid mime; `"Image Source Not Available"` if source missing.

***

## Examples

Resize to width 800, auto height:

```
https://img.actualize.earth/mybucket/photos/pic.jpg?width=800
```

Exact 600×500 cover crop:

```
https://img.actualize.earth/mybucket/photos/pic.jpg?width=600&height=500&crop=1
```

Square fit (no crop), grayscale:

```
https://img.actualize.earth/mybucket/photos/pic.jpg?width=400&square=1&gray=1
```

Return base64 (useful for embedding):

```
https://img.actualize.earth/mybucket/photos/pic.jpg?width=300&base64=1
```

Bypass cache for a fresh render:

```
https://img.actualize.earth/mybucket/photos/pic.jpg?width=800&nocache=1
```

> If you intend to rely on `cropratio`, pass something like `cropratio=3:2` and either a `width` or `height` (or both) so the final size is well-defined.

***

## Upload Helper (S3 multi-size)

`upload($qs)` is an internal helper that:

- Accepts an input image (either uploaded or by `url`) and generates a set of variants, defaulting to `thumb` and `full`. Built-in presets in code:

  - `thumb` 100×100 crop
  - `display` 600×500 crop
  - `cover` 600×500 crop
  - `profile` 200×200 crop
  - `background` 600×320 crop
  - `page` 2550×3300 crop
  - `full` quality 90 (no explicit size)
  - `small` quality 60 (no explicit size)
- Uploads to S3 at `/{hash}/{size}.{ext}` and returns paths/metadata.
- Recognized inputs include `sizes` (CSV or array), `path` (prefix; defaults to `/links/`), and `url` (to fetch remote source) among others.

> This is **not** directly a public endpoint in this file; you’ll need a route that passes the `$qs` array here.

***

## Public Methods Summary (for router integration)

- `ImageResizer::render($path, $qs = null, $toHeaders = true)`  
  Main entry. Reads `$path`, merges allowed params from `$qs`/`$_GET`, determines sizing, performs GM operations, and streams the result (or returns base64 string if requested by caller using `base64Encode`).

- `ImageResizer::getGifFrame($url)`  
  Returns a JPEG path of the first frame (used internally when needed).

- `ImageResizer::_ver()` → `'1.1'`  
  Simple version helper.

- `ImageResizer::base64Encode($o)`  
  Encodes a rendered file at `$o['src']` to `data:image/*;base64,…`.

- `ImageResizer::generateQR($qs)` / `generateQR2($qs)`  
  Streams a PNG QR code to the client.

- `ImageResizer::upload($qs)`  
  Generates multi-size variants and uploads to S3.

- `ImageResizer::outputToHeaders($o)`  
  Handles ETag/Cache headers and binary output.

***

## Edge Cases & Concerns (things to harden)

1. **Param surface vs. cache key**  
   The cache key excludes `height`, `width`, `nocache`, and `hex`. If you add new transform params, ensure the cache key includes them (or you’ll serve mismatched cached images).

2. **`hex`/`rgb` ghosts**  
   They’re referenced in query parsing/ignore lists but not wired into any transforms here. If you plan colored overlays/filters, either remove the vestiges or implement them to avoid confusion.

3. **EXIF orientation**  
   Comments indicate orientation handled by `-auto-orient` flags; ensure those flags are present in all GM call sites.

4. **Security**

   - Remote fetch uses cURL with desktop headers and writes to `/tmp/imagecache`. Consider a whitelist or proxy to avoid SSRF and very large downloads.
   - Validate mime and file size limits **before** handing to `gm convert`.

5. **S3 upload error handling**  
   Upload assumes `phi::$conf` and the S3 client; add try/catch and surface structured errors to callers.

***

## Quick Test Matrix

- Resize only: `?width=1200`
- Strict cover crop: `?width=1200&height=630&crop=1`
- Ratio-crop: `?cropratio=1:1&width=800`
- Square fit (no crop): `?width=600&square=1`
- Grayscale: add `&gray=1`
- Cache bypass: add `&nocache=1`
- Base64 response: add `&base64=1`
- GIF: same as above—first frame will be used

If you want, I can also produce a minimal OpenAPI path doc for the public pieces (render + qr) and a README snippet for your repo.