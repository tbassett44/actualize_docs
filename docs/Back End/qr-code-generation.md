---
title: QR Code Generation
excerpt: '****HIGHLY RECOMMENDED TO USE JS MODULE FOR QR GENERATION*****'
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
## QR Code Endpoints (resizer.php)

There are two QR flows; both ultimately output a PNG to the response.

1. **PHP-QRcode direct** (`generateQR`):

- `content` — raw content to encode
- `url` — URL to encode (will be `urldecode`d)
- `url_64` — base64-encoded URL to encode

Examples:

```
//VERSION 1
https://img.actualize.earth/qr?content=hello-world
https://img.actualize.earth/qr?url=https%3A%2F%2Factualize.earth%2Fjoin
https://img.actualize.earth/qr?url_64=aHR0cHM6Ly9hY3R1YWxpemUuZWFydGgvam9pbg==
//VERSION 2
https://img.actualize.earth/qr2?url=https%3A%2F%2Factualize.earth%2Fjoin
https://img.actualize.earth/qr2?url_64=aHR0cHM6Ly9hY3R1YWxpemUuZWFydGgvam9pbg==
```

2. **Node-rendered QR** (`generateQR2`):

- `url` or `url_64` — same as above; it builds a signed render URL and calls Node (`node/qr.js`) to produce the PNG, then streams it.

NOTE

More settings are possible for version 2 of the qr code generator. Only url/url_64 is supported right now.  Additional settings can be added from the documentation of the library used here: <https://qr-code-styling.com/>