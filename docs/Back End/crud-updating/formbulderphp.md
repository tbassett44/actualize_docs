---
title: formbulder.php
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
# `formbuilder.php` — Backend Module Documentation

## High-level overview

`formbuilder.php` is the server-side engine that powers create/read/update/delete (CRUD) for anything described in your `schema.json`. It validates input against the schema, enforces permissions, runs declarative hooks (before/after validation/save), optionally graphs/joins related data on reads, and can export/iterate large datasets. 

***

## Endpoints

* save – Saves/Updates a record using schema-driven validation, permissions, hooks, cloning, dot-updates, and returns current, previous, and diff when applicable. (Details below.)
* load – Read with optional graph, keep, and raw controls; removes fields you’re not allowed to see per field-level readPermission. 
* delete – Delete a record if the caller has delete permission (see Permissions). 

***

## Request & response (typical)

**Body:**

```json
{
  "schema": "<schema-id-or-name>",
  "current": { "...fields..." },
  "previous": { "...optional previous for diff/optimistic..." },
  "_partial": true,       // optional: partial update
  "_clone": { ... },      // optional: see Cloning
  "dot": { "...": "..." } // optional: dot updates
}
```

**Response:**

```json
{
  "success": true,
  "current": { ...sanitized... },
  "previous": { ...if provided or loaded... },
  "diff": { "field": {"set": ..., "removed": ...}, ... }
}
```

The `diff` is computed by comparing `previous` to `current`, with special handling for arrays (add/remove) and simple values (set/removed). Private fields are excluded from the diff.  

### Load

Supports:

* `graph`: apply schema-defined joins; or `"none"` to return raw docs.
* `keep`: explicit field allowlist (e.g., only what the front-end needs).
* `raw=1`: return raw (no graph) for JSON-editable content.\
  Field-level **readPermission** is enforced before returning. 

***

## Permissions

* **Module access**: Requests go through `handleRequest`, which dispatches to module methods. 
* **Edit permission**: `hasEditPermission` checks if the requester can write; used by `save`. 
* **Delete permission**: `delete` path verifies the caller’s scope/ownership before deletion. 
* **Field-level read permission**: `ensureDataByFieldPermissions` removes fields from the response when the caller lacks `readPermission`. 

> Tip: You can encode `readPermission` expressions in `schema.json` to hide sensitive fields from non-privileged users. 

***

## Validation & sanitization

Before persisting, `save` wires in the central `VALIDATOR` (see `classes/validator.php`) to type-check, coerce, and sanitize fields per schema control (e.g., `int` bounds, `url` normalization, allowed `select` options, HTML safety).    

Notable built-ins:

* `int` with `form.min/max` and optional money conversion. 
* `email` format checking. 
* `url` normalization and validation (with opt-out via `dontCheckURL`). 
* Safe text/HTML types: `text`, `textarea`, `html`, `html_template`, `text_basic`.  

***

## Save lifecycle

1. **Identify schema & API module**\
   `getApi` resolves a schema’s `module` to a PHP class for custom behaviors. 

2. **New vs. update**\
   If creating, `save` will generate a unique `id` per schema `create` rules (e.g., prefix/length), using uniqueness checks. 

3. **Hooks (see next section)**\
   `runHooks` executes your declarative hooks in order across lifecycle phases (`onBeforeValidation`, `onBeforeSave`, `onCreate`, `onUpdate`, `onAfterSave`, etc.). 

4. **Validation/sanitization**\
   Runs the `VALIDATOR` against `schema.fields` types and field constraints. (See above.) 

5. **Persist**\
   Saves via the data layer; on new records, fires `onCreate`; on edits, `onUpdate`. 

6. **Cloning (optional)**\
   If `_clone` is present and the schema has `onClone.copy`, related documents can be duplicated (multi or single), optionally clearing IDs/fields and merging overrides. Cloning uses internal `save` calls to create the children.   

7. **After-save**\
   Fires `onAfterSave`, broadcasts channel signals for `onCreate|onUpdate|onDelete`, and may trigger custom module methods. 

8. **Return**\
   Optionally re-loads via schema `loadMethod` or `route` and returns `current`, `previous`, and a computed `diff`.  

***

## Partial updates & unsetting

* **Partial**: If `_partial=1`, only the supplied fields are validated/merged; others remain untouched. 

* **Unset**: Set a field to the sentinel string `"[unset]"` to remove it. Arrays can use `["[unset]"]` semantics per diff/removal logic. (Be mindful of required/system fields.)  

* **Dot-updates**: For nested updates, provide a `dot` object (`{"a.b.c": <value>}`). The saver flattens & applies dot-paths. 

***

## Hooks

### Where hooks come from

Hooks are declared in `schema.json` under top-level lifecycle keys (e.g., `onBeforeValidation`, `onAfterSave`), or under individual **fields**, and are executed via `runHooks`. If a schema specifies `module`, methods are looked up on that PHP class; if not found there, built-in integrated hooks run.  

### Built-in integrated hooks (selected)

* **`ensureTags`** – Normalize & ensure tag references are persisted (e.g., add tag IDs to a collection). 
* **`ensureLocation`** – Normalize geolocation fields (id/place/city/state…). 
* **`processMedia`** – Post-process media payloads (images/videos). 
* **`sign`** – Set/validate signer metadata against the current user. 
* **`updateCMSChildren`** – Cascade updates for CMS folder children. 

If a schema’s `module` implements a method with the same hook name, that method is invoked; otherwise the integrated version runs.  

### Example hook declarations (from `schema.json`)

* **Tags**:

  ```json
  "onBeforeValidation": { "tags.add": { "hook": "ensureTags", "collection": "tag", "dataField": "tags" } }
  ```

  Ensures new tags exist and references are stable prior to validation. 
* **Location**:

  ```json
  "onBeforeValidation": { "location": { "hook": "ensureLocation" } },
  "onAfterSave": { "location": { "hook": "ensureCity" } }
  ```

  Normalizes address/geodata before validation and enriches after save. 

***

## Loading, graphing & “keep”

* **Graphing**: `load` can populate related docs according to the schema’s `graph` map (e.g., join admins, reviews, place by id).  
* **Keep list**: `keep` limits returned fields to a whitelist per schema section. Useful for fast lists. 
* **Raw**: supply `raw=1` to bypass graphing/formatting (e.g., WYSIWYG/JSON editors). Field‐level `readPermission` filtering still applies. 

***

## Iterate / export (feeds)

`iterate()` streams paginated results, optionally graphs them, and can export CSV with headers on page 0; it also supports S3 multipart uploads with progress. Useful for large admin exports.  

***

## Cloning related data

Schemas can declare `onClone.copy` to replicate related collections when an entity is cloned, with options to:

* `multi` — copy many related docs
* `clearId` — drop IDs in copies
* `merge` — override fields in clones
* `clear` — remove unwanted fields

The engine invokes `save` recursively for each clone item.   

***

## Diff & change tracking

After save, the module generates a structured `diff`:

* For scalar changes: `{"field":{"set":<new>}}`
* For array removals: `{"field":{"removed":[...]}}`
* For deleted fields entirely: `{"field":{"removed":true}}`\
  Private fields (flagged in schema) are excluded from diffs.  

***

## Error handling & retries

* Network/API failures surface meaningful `error` strings to the client; admin logging tracks unknown states. (See load/save callers in front-end `formbuilder.js`.) 

***

## Practical usage patterns

* **Create**: Provide `schema`, `current`; omit `id`. The engine generates an `id` per schema `create` rules and runs `onCreate` hooks.  
* **Update**: Provide `id` and changed fields in `current`; optionally pass `previous` to get a precise `diff`. Use `_partial` for patch-style updates.  
* **Unset**: Put `"[unset]"` in a field to remove it. 
* **Dot update**: Supply a `dot` map for nested paths. 
* **Read**: Use `load` with `graph`/`keep` or `raw`. Field-level `readPermission` is enforced automatically. 

***

## Gotchas & hardening suggestions

* **Required +`[unset]`** : Consider server-side guardrails preventing unsetting fields marked `required` or `system` in the schema. (Unset is powerful.) 
* **Partial validation**: With `_partial`, ensure hooks that depend on other fields are resilient when those fields are absent. (Place them in `onBeforeSave` instead of `onBeforeValidation` when appropriate.) 
* **Read-permission leaks**: Double-check `graph`ed subdocuments for sensitive fields; the engine strips by field, but downstream joins/templates should avoid re-exposing restricted data. 
* **Clone blasts**: `onClone.copy` can recursively create many records—add quotas/owner checks where needed to avoid accidental data explosion & unexpected costs. 
* **Export throughput**: CSV+S3 uploads run in long loops; monitor timeouts and set sensible page sizes to keep infra cost in check.  

***

## Cross-reference: declaring forms & fields

Front-end `formbuilder.js` consumes the same `schema.json` entries (types, form options, options lists, toggle rules, etc.) for UI. Example `money`, `textarea`, `tag`, and graph rules are visible in the sample schema.
