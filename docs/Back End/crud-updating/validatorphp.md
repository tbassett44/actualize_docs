---
title: validator.php
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
# `validator.php` — Validation & Sanitization Engine

**What it does (at a glance):** Centralizes data validation/sanitization for objects described by your `schema.json`. Given an input `$data` and a schema `$conf`, it normalizes values, enforces required/unique rules, validates types (including nested arrays/objects), and returns either a **cleaned payload limited to the schema’s`order`** or a structured error report.  

***

## Public API

### `VALIDATOR::validate($data, $conf, $update, $dont_allow_system_updates=false, $last=false)`

Runs the full pipeline:

1. **Skip private/system fields** (they’re removed from `$data`). 
2. **Per-field processing**: determine type config (`$tconf`), handle passwords (optional confirmation), auto-create `id/_id/timestamp`, then sanitize/format values.   
3. **Uniqueness check** (with optional `unique.updateOn` allowance for self-updates). 
4. **Required rules**: different behavior for **create** vs **update**; consciously allows `false` for `bool` and `0` for `int/float`.  
5. **Malformed check** via `checkData()` and `isValidDataType()`. 
6. **Return**:

   * On error → structured `['error' => …, 'schema' => …]`. 
   * On success → **only** fields listed in schema `order` (or pass-through when `jsonEditable`). 

### `VALIDATOR::keepFields($arr, $fields)`

Returns `$arr` limited to `order`, including support for **dot-prefix matching** of nested keys. 

***

## Supported Types

Primitive & composite types (with special behavior):

| Type                     | Meaning / Rules                                                                                                                                     |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `string`                 | Cast to string, **strips all HTML** unless `allowTags` is set.                                                                                      |
| `string_unsafe`          | Cast to string, **no tag stripping**.                                                                                                               |
| `int`                    | Must be `is_int` after formatting; supports bounds and form min/max. `0` allowed.                                                                   |
| `float`                  | `is_float`; optional `round`; supports bounds. `0.0` allowed.                                                                                       |
| `bool`                   | Accepts `true/false` and strings `'true'/'false'`.                                                                                                  |
| `email`                  | Lower-cased; validates with `FILTER_VALIDATE_EMAIL`. Blank allowed.                                                                                 |
| `url`                    | `ensureURL()` prefixes `https://` if missing; then URL validation (see caveats). Can be disabled per field via `_validateOpts[field].dontCheckURL`. |
| `password`               | When `$hashPassword` is true (default), hashes as `md5($val . AUTH_SALT)`. Supports **confirmation** via `validate` sibling field.                  |
| `html`                   | Sanitized with `phi::ensureRedactorContent` and an HTML Purifier instance.                                                                          |
| `html_template`          | Accepted as-is (string).                                                                                                                            |
| `text` / `textarea`      | Stored as string; HTML stripped.                                                                                                                    |
| `text_basic`             | String with limited `allowTags` (`<b><i><span><strong>`).                                                                                           |
| `hexcolor`               | Must match `#rgb` or `#rrggbb`.                                                                                                                     |
| `imageextension`         | Must be one of `png`, `jpg`, `jpeg`, `webp`.                                                                                                        |
| `image`                  | Object with fields: `ext`, `path`, optional `ar` (float), `v` (int).                                                                                |
| `link`                   | Object: `{ url, type }`.                                                                                                                            |
| `links`                  | Array of `link`.                                                                                                                                    |
| `multiimage`             | Array of `image`.                                                                                                                                   |
| `object`                 | Nested object; subfields validated recursively (supports per-field `notrequired`).                                                                  |
| `array`                  | Array of `itemType`; validates each element; enforces `form.max` if present. Special case: a single string `['false']` is treated as unset to `[]`. |
| `point`                  | `{ type: string, coordinates: coords }`.                                                                                                            |
| `coords`                 | `{ 0: float, 1: float }`.                                                                                                                           |
| `geotext`                | `{ main: text, secondary: text }`.                                                                                                                  |
| `geocode`                | `{ lat: latlng, lng: latlng, place_id?: string, text?: geotext }`.                                                                                  |
| `latlng`                 | Float (used inside `geocode`).                                                                                                                      |
| `_id`, `id`, `timestamp` | Strings/ints; may be **auto-created** on create when `create` is set in the field config.                                                           |
| `redactor`               | Alias to `html`.                                                                                                                                    |
| `index`                  | Present but not defined (reserved/unused).                                                                                                          |

> **Select fields:** If the schema sets `form.type: 'select'` with `options.order`, the chosen value must be one from that `order`. 

***

## Formatting vs. Validating

* **`formatData()`** → coercion & sanitization (e.g., strip tags, lowercase emails, add `https://`, hash passwords, round floats, recursive handling for arrays/objects). 
* **`checkData()`** → structural & type validation; delegates to `isValidDataType()` for primitives; recurses for arrays/objects. 
* **`isValidDataType()`** → primitive checks + extras (bounds, min/max, regex for hex color, select-option enforcement, URL/email rules). 

***

## Required, Defaults & Special Flags

* **Required logic**:

  * On **create** (`$update=false`): missing/blank fails except for `bool=false` and numeric `0`. 
  * On **update** (`$update=true`): run required checks **only** for fields present in input; same `false/0` allowances apply. 
* **`create`**: auto-fill `id/_id` (GUID) or `timestamp`. 
* **`dontAllowZero`**: after formatting, if value is `0`/falsy, the field is **unset** (useful when zero is semantically invalid). 
* **`_validateOpts[field].dontCheckURL`**: bypass live URL validation for that field. 
* **Unset sentinel**: sending the literal string `'[unset]'` attempts to unset a field; disallowed if the field is required. 

***

## Unique Constraints

* Add `{ unique: true }` to a field to ensure uniqueness across the collection.
* To allow updates when the “same record” is being modified, use `unique.updateOn` with a **dot path** that must equal in both the incoming `$data` and the existing row. On match, the update is allowed. 

***

## Error Shape

When validation fails, `validate()` returns:

```json
{
  "error": {
    "Missing Data": [{ "type": "...", "name": "...", "data": <value>, "field": "..." }],
    "Malformed Data": [{ "type": "...", "name": "...", "data": <value>, "field": "..." }],
    "Already Being Used": [{ "type": "...", "name": "...", "data": <value>, "field": "..." }],
    "Invalid": [{ "type": "...", "name": "...", "field": "..." }]
  },
  "schema": "<schema id>"
}
```

***

## Example (Schema Snippet)

```json
{
  "id": "user",
  "order": ["id", "email", "name", "avatar", "website"],
  "fields": {
    "id": { "type": "id", "create": "user" },
    "email": { "type": "email", "required": true, "unique": true },
    "name": { "type": "string", "required": true, "form": { "type": "text" } },
    "avatar": { "type": "image", "notrequired": true },
    "website": { "type": "url", "form": { "type": "text" } }
  }
}
```

* On create, `id` is generated; `email` must be unique; `website` auto-prefixed with `https://` if missing.   

***

## Gotchas & Recommendations (robustness / security)

* **Password hashing (`md5`)**: MD5 is not suitable for passwords. Prefer `password_hash()`/`password_verify()` (bcrypt/Argon2) and store a per-user salt; keep `$hashPassword` for tests/seeding only. 
* **URL validation side-effects**: `isValidUrl()` may perform a network request (`get_headers`) and even contains a `die(json_encode($headers))` code path—this can terminate execution unexpectedly and risks SSRF. Strongly consider removing remote fetches or constraining them via allowlists/timeouts. 
* **`ensureURL()`** always forces `https://` when no scheme—great default, but may reject valid non-HTTP(S) schemes you might want (e.g., `mailto:`, `tel:`, `ipfs://`). Consider a scheme allowlist per field. 
* **`imageextension`case-sensitivity** : allowed list is lower-case; ensure upstream normalization (or lower-case here) to avoid false negatives. 
* **`index`type** is empty—clarify or remove to avoid confusion. 
* **Sentinel`'[unset]'`** : convenient, but be cautious exposing it directly to clients; consider explicit patch semantics for clarity. 

***

## Implementation Notes

* **Formatting is applied before validation** (`formatData()` then `checkData()`), so constraints (e.g., `min/max`) evaluate on normalized values.  
* **Select fields**: ensure your `schema.json` has `form.options.order` enumerated; otherwise valid values will be rejected. 
* **Output field limiting** via `order` ensures only **validated** fields flow downstream. For JSON-editable documents (`jsonEditable`), the raw (formatted) data is returned. 

***

## Quick Reference (flow)

1. Strip private/system → type lookup → (optional) create defaults → format → uniqueness → required → validate → **`order`filter / jsonEditable** → return.
