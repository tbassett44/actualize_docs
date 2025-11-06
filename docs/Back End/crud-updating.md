---
title: CRUD updating
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
# Backend Schema & Hooks — How data is defined, validated, saved, and extended

This documents how your backend **schema.json** drives data modeling, server-side validation/sanitization, and lifecycle hooks that run around saves. It also clarifies where the client hands off to the server and what happens next.

***

## 1. Where the request comes from (client → server)

Your FormBuilder posts only the **current field values**, the **schema type (collection name)**, and **timezone** to the save endpoint:

### Client Side

```js
modules.api({
  url: app.sapiurl + '/v1/schema/[schema_id]',
  data: {
    current: {},
    timezone: _.getTimeZone()
  },
  timeout: 5000
});
```

That means **the server uses schema.json** to validate/sanitize based on the `schema` string, *not* on a client-passed shape. 

### Server Side

<br />

***

## 2. What a “Schema” looks like (collections, fields, and UI hints)

A typical collection entry in `schema.json` includes:

* **Collection-level metadata**: `name`, `module`, `module_path`, `roles`, `order`, `compoundIndexes`, and optional `graph` (server-side population rules). Example: the `event_promotion` collection declares `module_path`, `module`, `roles`, an `order` array, and a compound index on `start,end`. 

* **Fields**: Each field has a `type`, `required`, `name`, optional `_index`, and a **`form`config** (used by the FE generator but also helpful to understand constraints).\
  Example field types you use here: `id`, `text`, `timestamp`, `object`, `image`, `point`, `tag`, `int`, `string`, etc. The `point` field uses a **geospatial index**: `"_index": "2dsphere"`. 

* **Creation helpers**: `id` fields often include a `create` block with `len`, `pre`, and optional **uniqueness constraints** (collection/table/field) enforced server-side. 

* **Permissions / privacy**: Per-field restrictions like `permission: ["admin"]` and flags like `"private": true` exist (e.g., `_id` is private in `email_broadcast`).  

* **Graph joins**: Collection-level `graph` defines how the backend materializes references (which collection to join, which field to match, and where to put the result). Example: `signal_group.graph.threads` and `email_broadcast.graph` entries.  

* **Channels**: Some collections define outbound “channels” with a `template` and `variables` (e.g., email campaigns). 

***

## 3. Validation & Save Lifecycle

From `schema.json` you define **lifecycle hooks** that the backend calls at specific phases. In practice, saves flow like this:

1. **onBeforeValidation** → mutate/enrich incoming values so validation passes (e.g., denormalize, set derived fields)
2. **Validation & sanitization** (types, required, indexes/uniques, domain rules)
3. **onBeforeSave** → last chance to transform, block, or prepare side effects
4. **Write to DB**
5. **onAfterSave** → launch side effects that depend on the stored record (enqueue jobs, schedule, etc.)

You use all three hook phases extensively:

* **onBeforeValidation** — normalize inputs so the later validators have the right shape

  * `location` fields call `ensureLocation` to fill geodata and project to a `point` field. 
  * `birthday` calls `setBirthdayTime` to set a derived timestamp (and sign/month/day/year). 
  * `tags` call `ensureTags` to resolve/insert tags and attach `*_info` lists. (Used for `tags`, `skills`, `event` tags, etc.) 

* **onBeforeSave** — prepare data, enforce business rules, or compute IDs

  * `event_promotion.id` runs `preProcessPromotion` before commit. 
  * Other examples include checking contacts before saving and similar gating logic. 

* **onAfterSave** — trigger actions that should run only once the record exists

  * Scheduling future sends for `email_broadcast.time` (`scheduleSend`) and a guard `checkSend`. 
  * Post-save game mechanics such as `checkBirthdayGame` and `checkSkillsGame`. 
  * Creating QR codes or downstream artifacts (`initiateSignalQR`). 

<Callout icon="🔎" theme="default">
  ### **Client/server boundary:** because FormBuilder submits only `schema` + `current`, *all* these decisions are driven server-side, which is good for integrity and permissioning.
</Callout>

***

## 4. Hook catalog (from the schema)

Below is a non-exhaustive list of hook names you’ve defined in `schema.json` (grouped by phase) with examples of where they appear.

| Phase                | Hook name             | Purpose (in practice)                              | Example location(s)                         |
| -------------------- | --------------------- | -------------------------------------------------- | ------------------------------------------- |
| `onBeforeValidation` | `ensureLocation`      | Resolve Mapbox result into `location.info`/`point` | `location` objects in multiple collections. |
| `onBeforeValidation` | `setBirthdayTime`     | Compute `birthday.ts` and related derived fields   | `user` birthday object.                     |
| `onBeforeValidation` | `ensureTags`          | Create/resolve tag docs; attach `*_info` fields    | `tags`, `skills`, `event` tag fields.       |
| `onBeforeValidation` | `eventStartUnique`    | Enforce uniqueness of start timestamps for events  | Event date fields.                          |
| `onBeforeSave`       | `preProcessPromotion` | Pre-commit processing of promotion IDs & fields    | `event_promotion.id`.                       |
| `onBeforeSave`       | `checkContact`        | Block or update based on contact state             | Contact-related save.                       |
| `onAfterSave`        | `scheduleSend`        | Schedule outbound email campaign                   | `email_broadcast.time`.                     |
| `onBeforeSave`       | `checkSend`           | Guard against invalid scheduling states            | `email_broadcast.time`.                     |
| `onAfterSave`        | `checkBirthdayGame`   | Trigger gamified behavior after profile updates    | `user` birthday object.                     |
| `onAfterSave`        | `checkSkillsGame`     | Post-save skills gamification                      | `user.skills`.                              |
| `onAfterSave`        | `initiateSignalQR`    | Generate/share QR for Signal flows                 | Contact/Signal setup.                       |

> You also use collection-level configuration such as `graph` to populate referenced data (e.g., `signal_group.threads`, `email_broadcast.*`) and form hints like `form.type: "timezone" | "date" | "tags" | "location"` that drive front-end behavior but align with server expectations.  

***

## 5. Patterns worth calling out (design guidance)

* **Keep mutations in`onBeforeValidation`, not in validators**\
  Your current use (e.g., `ensureLocation`, `ensureTags`, `setBirthdayTime`) is ideal: derive canonical values *before* validation rules run, so validators operate on stable shapes.  

* **Idempotency & retries for`onAfterSave`**\
  Hooks like `scheduleSend`, `initiateSignalQR`, and game triggers should tolerate duplicate delivery (e.g., via idempotency keys) because clients may retry saves or servers may re-run jobs after failures.  

* **Put heavy effects on a queue**\
  Anything that schedules, hits third-party APIs, or creates images should enqueue a job rather than run inline; it reduces tail latency and avoids timeouts. (You’re already time-boxing client calls at 5–8s in other modules.)

* **Geospatial correctness**\
  You’re generating a `point` with a `2dsphere` index—great for `$near` queries. Make sure `ensureLocation` always writes **\[lng, lat]** ordering (it does today) and normalizes precision to reduce noisy duplicates. 

* **Uniqueness enforcement**\
  You use both explicit uniqueness in `create.unique` and logical uniqueness hooks (e.g., `eventStartUnique`). Prefer **DB-level unique indexes** (where feasible) to ensure correctness under concurrency; back them up with hook checks for clear UX.  

* **Security & permissions**\
  Respect field-level `permission` arrays on the server, not just in the UI. Avoid trusting `form` settings from clients; treat them as hints only. (E.g., `city_steward.permission: ["admin"]`, `_id.private: true`.)  

* **Graphs are read-model helpers**\
  Because `graph` drives server-side hydration, keep it **deterministic** and **side-effect free**. If a join can explode cardinality, use `collapseList: true` like you do today. 

* **Clock & timezone safety**\
  Date fields often bind to a `timezoneField` (e.g., `timezone`); that’s excellent. Ensure the server resolves all timestamps using that field to avoid drift (and validate that `start <= end`). 

* **Cost/throughput considerations (finance minded)**

  * **Email scheduling** and **QR/image generation** can create bursty spend. Rate-limit `onAfterSave` operations and centralize retry with exponential backoff.
  * Caching for `ensureTags`/`ensureLocation` avoids re-creating the same entries, reducing DB writes and API costs over time. 

***

## 6. Quick reference: schema building blocks you use

* **Collection metadata**: `name`, `module`, `module_path`, `roles`, `order`, `graph`, `compoundIndexes`.  
* **Field core**: `type`, `name`, `required`, `_index`, `permission`, `private`, `create`, `hooks`.  
* **Form hints** (front-end): `form.type`, `placeholder`, `template`, `endpoint`, `info`, `timezoneField`, `toggle`, etc. (Good for FE, but server must not trust them.)  
* **Graph joins**: `coll`, `to`, `match`, `collapseList`. 

***

## 7. Example: location object with hooks (end-to-end)

```json
"location": {
  "type": "object",
  "required": true,
  "fields": { "id": { "type": "string" } },
  "types": ["place"],
  "form": { "type": "location", "info": "location.data" },
  "hooks": {
    "onBeforeValidation": {
      "ensureLocation": { "key": "point", "info": "location.data" }
    }
  }
}
```

* FE renders a location picker, stores both the chosen place and a projected **`point`**.
* Server runs `ensureLocation` **before validation** to guarantee geo coherence and attach `location.data`, then validates, saves, and any `onAfterSave` hooks run. 

***

## 8. What to add next (recommendations)

* **Formalize hook interfaces**: Document required params & return contracts for each hook (`ensureTags`, `ensureLocation`, etc.) and make them **pure functions** (inputs → outputs), so they’re testable and cost-predictable.
* **Central error taxonomy**: Your FE already normalizes errors via `modules.formbuilder_global.getError`. Mirror that shape in hook errors (e.g., `type`, `field`, `message`) so the UI can highlight fields precisely. 
* **Audit & replay**: Record hook decisions (e.g., why an email was scheduled) for accountability and future reprocessing.
* **DB constraints**: Back critical invariants with DB indexes (unique, partial), especially for promotions and event times.
