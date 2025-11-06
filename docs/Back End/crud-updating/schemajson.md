---
title: schema.json
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
# `schema.json` — Collection & Hook Specification

This document explains how your `schema.json` defines backend collections, field metadata, relationships, hooks, indexes, permissions, and UI hints that the FormBuilder + validator use at runtime. Where useful, I cite concrete lines from the uploaded `schema.json` (and backend handlers) so you can verify each behavior.

***

## 1. What a “collection schema” looks like

Each top-level key is a collection with optional metadata such as `name`, `order`, `roles`, `graph`, `channels`, `module`, `module_path`, `compoundIndexes`, and the required `fields` map.

**Examples**

- `tags_post` declares a display name, a field display order, and a `fields` map.  
- `email_broadcast` shows `keyOn`, UI channels, role restriction, and graph lookups.   
- `event_promotion` includes business text (guidelines), a canonical field order, graph joins, role restriction, module routing, and a compound index.   
- Other global flags you’ll see across collections: `jsonEditable` (freeform JSON editing) and `scopes` (token write scopes).  

***

## 2. Fields: types, creation, indexing, and form hints

A field entry typically contains:

- `type` (e.g., `id`, `text`, `object`, `timestamp`, `point`, `image`, `tag`, `html_template`, etc.)
- `name` (human label), `required`, `_index` (for DB indexing), and optional `create` options for IDs.
- `form` block providing UI rendering hints for FormBuilder.

**Common patterns**

- **ID fields** with auto-create rules (length + prefix). 
- **Simple text** with textarea form and constraints. 
- **Timestamps** with timezone-aware date pickers.  
- **Tag fields** with a tags UI + server-side tag assurance.  
- **Phone object** (code/number/iso2) with specialized `phone` form type.  
- **Images** with crop/display presets. 
- **Email editors** (`email_editor`, `email_editor2`) that manage text+HTML fields and uploads.  
- **Toggle-controlled UI** (e.g., switch between template v1/v2 by turning fields on/off).  

**Indexing**

- `_index: 1` for standard indexes; compound indexes supported at the collection level. 
- **Geo**: a `2dsphere` index is used for geospatial queries on a point field. Note the schema uses both `type:"object"` for a `point` field in some places and `type:"point"` in others—pick one convention and stick to GeoJSON to avoid validator/driver mismatches.  

**Uniqueness**

- IDs can declare remote uniqueness enforcement (`unique` with `{collection, table, field}`). 

***

## 3. Relationships: `graph` lookups

Use `graph` to hydrate references from other collections into the current record, optionally selecting specific fields and collapsing lists.

- Example in `email_broadcast`: look up tags and campaign by `id`, with `collapseList` and `fields` selection.  
- Example in `event_promotion`: join `user`, `event`, `place`, `payment_info`, and `exchange`. 

At read time, `graph` (and `graph2`) are applied to expand records. 

***

## 4. Channels (push/broadcast)

Collections can define `channels` with a `template` name and `variables`. When a record changes, the backend publishes to those channels, substituting bracket variables from the record.  

***

## 5. Hooks: where to plug logic

You can attach hooks either **on fields** (most common) or handle collection-level change semantics via the backend. The primary lifecycle events observed are:

- `onBeforeValidation` — transform/verify user input prior to validation (e.g., normalizing media, ensuring geocodes/tags).  
- `onBeforeSave` / `onBeforeCreate` — perform side effects or enrich data before persistence (e.g., pre-process, payment orchestration, ensure relationships).   
- `onAfterSave` — post-commit actions (e.g., kick off QR init). 

The server iterates field hooks for the current lifecycle `$type`, and there’s a commented list of recognized phases including `onCreate`, `onAfterSave`, `onAfterUpdate`, `onDelete`, `onBeforeCreate`. (You also see an `onUpdate` branch.)  

**Concrete hook helpers used in `schema.json`**

- `ensureLocation` — writes a GeoJSON point into a target field (`key`) and copies descriptive info from another path (`info`).  
- `processMedia` — normalizes media object(s) before validation. 
- `ensureTags` — resolves tag IDs to data and stores in a `dataField` for display/query. 
- `preProcessPromotion` and `processPayment` — business flows for promotions; can target fields like `payment.payment_id`, read `total`, and set descriptions.  
- `ensureSignalGroup` — integrity check for the thread’s parent group before save. 

***

## 6. Feed & query behavior that reads schema

The backend “feed” endpoint honors several schema directives:

- **`keyOn`** (e.g., `_id`, `tsu`, `id`) controls pagination key and default sorting; if `tsu`, sort by timestamp.  
- **`keep`** limits projected fields unless `showAllData` is set. 
- **Filters** can come from `schema.filter` (stringified JSON with token replacements like `[uid]`), `schema.query`, or a dynamic `getFilter` method on the module class.  

***

## 7. Access control & permissions

- **Collection-level** access: `roles` restrict who can use a collection’s admin/editor surfaces. (Example: `email_broadcast` is admins only.) 
- **Field-level** read filtering: optional `readPermission` on a field is enforced at read time; if the requester lacks scope, the server strips that field from the payload. 
- **Token scopes**: the `token` collection defines `scopes` for write operations. 

***

## 8. Location & geo patterns

- Location objects often limit types (e.g., `["place"]`), store a `location.id`, and carry a hidden `point` index for proximity queries. Use `ensureLocation` to maintain the GeoJSON point automatically.  
- Some feeds support a `geoloc` option that leverages `2dsphere` for distance queries (backend logic). 

***

## 9. UI-only metadata (used by FormBuilder)

Every field can include a `form` block that hints how to render/edit:

- Hidden fields for system-managed values (IDs, computed text, points).  
- Specialized editors (`email_editor`, `email_editor2`, `timezone`, `multiselect`, `tags`, `phone`, `onoff`, `image` crop presets).      

***

## 10. Worked examples

### A. Minimal “tag” collection (`tags_post`)

- Two fields: `id` (auto-generated) and `name` (required text).  
  The collection also defines how fields are ordered in views.  

### B. Promotion flow (`event_promotion`)

- Business copy, role restriction, module routing, compound index.  
- Field hooks:

  - `id.onBeforeSave.preProcessPromotion` (pre-work before save). 
  - A separate `onBeforeCreate.processPayment` block (prepare and charge, persist `payment.payment_id`). 
- Timezone-aware start/end date pickers; totals, payment graph joins.   

### C. Messaging modes (`email_broadcast`)

- Channel push integration; UI toggle to switch between two email editor modes (`email_mode` toggles `email_template` vs `email_html_message`).   

***

## 11. Gotchas & recommendations (logic check)

- **Standardize `point`**: Some collections model the geo field as `type:"object"` while others set `type:"point"`. Pick one (prefer `type:"point"` storing valid GeoJSON `{ type: "Point", coordinates:[lng,lat] }`) and ensure the `2dsphere` index references that exact shape. Mixed conventions risk validator or query mismatches.  
- **Hook clarity**: Document which lifecycle events your validators actually fire (`onBeforeValidation`, `onBeforeCreate`, `onBeforeSave`, `onAfterSave`, `onAfterUpdate`, `onDelete`, `onUpdate`). Keep hook names consistent with backend expectations.  
- **Field-level permissions**: If you rely on `readPermission` to hide sensitive data, test this path. The server strips fields when the caller lacks scope—good, but easy to overlook in UI if you assume presence. 
- **Unique ID generation**: When using `create.unique` with an external table, ensure the target collection/table/field actually enforce a uniqueness constraint to avoid race conditions at scale. 
- **Channel variables**: Keep channel templates and variable names in sync with your record’s shape; they’re replaced by bracketed keys like `[var]`. 

***

## 12. Quick reference checklist for adding a new collection

1. **Metadata**: `name`, optional `order`, `roles`, `module`/`module_path`, `keyOn`, `compoundIndexes`. (See `event_promotion` for a complete example.)  
2. **Fields**: define `type`, `required`, `form` hints, `_index` where needed, and `hooks` (e.g., `onBeforeValidation`).  
3. **Relations**: use `graph` to hydrate linked data; pick `fields` and `collapseList` thoughtfully. 
4. **Channels**: add `channels` if you need pub/sub updates on changes. 
5. **Permissions**: set `roles` and any field `readPermission` rules.  
6. **Geo**: if proximity search is relevant, include a `point` with `2dsphere` and an `onBeforeValidation.ensureLocation` hook.