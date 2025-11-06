---
title: Formbuilder
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
# High-level: What the Form Builder Module Does

The form builder reads your `schema.json` and **renders dynamic, data-bound inputs** using `formbuilder.templates`. Each element type defines a `render(...)`, `getCurrent(...)`, and `binding(...)` to (1) produce the correct template, (2) seed/resolve default values, and (3) wire events/update `options.current` state. Advanced elements call APIs (e.g., payment methods, preview computations), can autosave, and can write to related fields for denormalized storage (e.g., HTML/text twins, tag info maps). Uploaders and pickers handle permissions, progress, and clear/reset flows in place.

# All supported form types with brief description

| Form Type           | Description / Behavior                                                                                             |
| :------------------ | :----------------------------------------------------------------------------------------------------------------- |
| `text`              | Standard text input field.                                                                                         |
| `textarea`          | Multi-line text area, can support placeholder, maxlength, minHeight .                                              |
| `hidden`            | Hidden input for IDs, system fields, etc. .                                                                        |
| `id`                | Special identifier field, often auto-generated with prefixes.                                                      |
| `date`              | Date/time picker, often linked to timezone and supporting formats (e.g. `event` format, linked start/end fields) . |
| `timestamp`         | Stores full timestamp values (often rendered as `date`).                                                           |
| `timezone`          | Dropdown or selector for timezone values .                                                                         |
| `onoff`             | Boolean toggle (true/false switch)  .                                                                              |
| `buttonselect`      | Button group selector with named options (supports order, list of values, descriptions)  .                         |
| `tags`              | Tagging input with API-backed search for tags, templates for display .                                             |
| `color`             | Color picker widget using iro.js, supports hex input【85:2†formbuilder.js†L:contentReference[oaicite:0]{index=0}    |
| `email_link_tags`   | Complex object to associate tags with links inside emails .                                                        |
| `module`            | Embeds or references another module.                                                                               |
| `select`            | Standard dropdown select input.                                                                                    |
| `multiselect`       | Multi-select dropdown.                                                                                             |
| `number` / `int`    | Numeric input (integer).                                                                                           |
| `float`             | Numeric input (floating-point).                                                                                    |
| `checkbox`          | Boolean via checkbox.                                                                                              |
| `radio`             | Radio button group for single-choice selections.                                                                   |
| `file`              | File uploader (integrates with file/image uploaders).                                                              |
| `image`             | Specialized file/image uploader with preview.                                                                      |
| `richtext` / `html` | Rich text editor (Redactor-based), supports AI, images, variables:contentReference[oaicite:1]{index=1}.            |
| `password`          | Password field (hidden input).                                                                                     |
| `currency`          | Numeric field formatted as currency.                                                                               |
| `percent`           | Numeric field formatted as percentage.                                                                             |
| `range`             | Slider/range selector.                                                                                             |
| `button`            | Generic action button.                                                                                             |
| `jsonEditable`      | JSON editor field (raw data editing).                                                                              |

# Form Builder — Supported Form Elements (Reference)

Below is a concise, copy-pasteable reference for every form element type I could find across your `formbuilder.js`, `formbuilder.templates`, and `schema.json`. Each section explains what the element does, the key options you can pass under `field.form`, notable behaviors, and a minimal example. I also included a short, high-level overview of the module at the end.

***

## Text / Textarea

**What it does:** Renders a plain text input rendered as a textarea. Often used for short titles or one-liners with optional limits.  
**Key options:**

- `type: "textarea"` — render as textarea.
- `placeholder`, `maxlength`, `minHeight`, `default`, `singleLine` (when used in rich-text variants).   

**Example:**

```json
"email_subject": {
  "type": "text",
  "name": "Email Subject",
  "required": true,
  "form": { "type": "textarea", "maxlength": 100, "minHeight": 30 }
}
```

***

## Rich Text (Redactor)

**What it does:** Rich-text / HTML editor with plugins, variables, and media uploads. Also supports “plain text only” when `nohtml: true`.  
**Key options:**

- `type: "redactor"`
- `plugins` (e.g., `"variable"`), `variables` (token list with `[]`), `buttons`, `minHeight`, `maxlength`, `singleLine`, `nohtml`, `placeholder`, `default`
- `uploadOpts.image|file` endpoints
- Live syncing to sibling fields via `htmlField` (full HTML) and `textField` (plain-text).
- Autosave: `autosave: { versionField: "…" }`
- Editor stream/AI config wired internally.    

**Example:**

```json
"message": {
  "type": "html",
  "name": "Message",
  "required": true,
  "form": {
    "type": "redactor",
    "maxlength": 10000,
    "minHeight": 30,
    "plugins": ["imagemanager_nectar","button"],
    "uploadOpts": {
      "image": { "endpoint": "[apiurl]/upload/image/submit", "opts": {} },
      "file":  { "endpoint": "[apiurl]/upload/file/submit",  "opts": {} }
    },
    "variables": ["[to.data.name]","[event.name]"]
  }
}
```

***

## Email Editor

**What it does:** Specialized editor for email content/blocks with support for variables and programmatic actions.  
**Key options:**

- `type: "email_editor"` (template: `formbuilder_email_editor`)
- Supports actions such as variable injection/clear via `handleAction`.  

***

## Code Editor (ACE)

**What it does:** Inline ACE editor for code/json with mode selection.  
**Key options:**

- `type: "code_editor"` (template: `formbuilder_code_editor`)
- `mode` (e.g., `ace/mode/json` auto-defaults to `"{}"`), `default`.  

***

## Button Select (Pills / Segmented)

**What it does:** Renders a segmented set of choice buttons with labels and descriptions.  
**Key options:**

- `type: "buttonselect"`
- `default`, `keepPlaceholder`, `options.order`, `options.list[{ value, name, description }]`. 

**Example:**

```json
"stop": {
  "type": "string",
  "name": "Stop Ticket Sales",
  "form": {
    "type": "buttonselect",
    "default": "end",
    "options": {
      "order": ["end","day_before","hour_before","start"],
      "list": { "end": { "value": "end", "name": "At End of Event" } }
    }
  }
}
```

***

## Tags (Remote Search/Select)

**What it does:** Search & select one or more tag records (with optional extra info caching).  
**Key options:**

- `type: "tags"` or `"tag"`
- `endpoint`, `endpointOpts` (e.g., `{ collection: "role" }`), `template` (e.g., `formbuilder_tag_page`), `icon`, `placeholder`, `info` (where to store fetched info).  
- Selected IDs and optional `info` map stored into `current`. 

**Example:**

```json
"roles": {
  "type": "tag",
  "name": "Roles",
  "form": {
    "type": "tags",
    "endpoint": "[apiurl]/search/tags",
    "template": "formbuilder_tag_item_plain",
    "endpointOpts": { "collection": "role" },
    "info": "role_info"
  }
}
```

***

## Page / Entity Picker

**What it does:** Opens a page-style picker (aggregate search) to select a user/page/etc.  
**Key options:**

- `type: "page"`
- `template` for item view (`formbuilder_tag_page`), `endpoint` (aggregate), `endpointOpts.filters` and flags like `allowMe`. 

**Example:**

```json
"reply_to": {
  "type": "object",
  "name": "Reply To",
  "required": true,
  "form": {
    "type": "page",
    "placeholder": "Reply To",
    "template": "formbuilder_tag_page",
    "endpoint": "[apiurl2]/search/aggregate",
    "endpointOpts": { "filters": ["people"], "allowMe": true }
  }
}
```

***

## Location (Place Autocomplete + Geo Hooks)

**What it does:** Structured place selector with `id` and optional info hydration; used with geo hooks to set a `point`.  
**Key options:**

- `type: "location"` (object with `fields.id`)
- `types` (e.g., `["place"]`), `nomore`, `info` target for fetched details.
- Pairs with hook `ensureLocation` to derive/store geo point.  

**Example:**

```json
"location": {
  "type": "object",
  "name": "Location (city)",
  "types": ["place"],
  "fields": { "id": { "type": "string" } },
  "form": { "type": "location", "info": "location.data" },
  "hooks": { "onBeforeValidation": { "ensureLocation": { "key": "point", "info": "location.data" } } }
}
```

***

## Phone (Composite)

**What it does:** i18n phone input composed of code/number/iso2; rendered inline.  
**Key options:**

- `type: "phone"` on `form` for an object with fields `code`, `number`, `iso2`; `inline: true`. 

***

## Date / Time (Timestamp)

**What it does:** Timestamp input with event-aware formatting and timezone coupling.  
**Key options:**

- `type: "date"` on `form` for a `timestamp` field.
- `format: "event"`, `timezoneField`, and `linkedTo` to constrain end vs. start.  

***

## Timezone

**What it does:** Timezone selector; often referenced by date fields with `timezoneField`.  
**Key options:**

- `type: "timezone"`. 

***

## Hidden

**What it does:** Stores internal/system values not shown to users.  
**Key options:**

- `type: "hidden"`; commonly used for IDs, derived geo points, or system flags.  

***

## Image

**What it does:** Image picker/uploader with display presets and cropping modes.  
**Key options:**

- `type: "image"`; `multiple`, `display: "background"`, `crop: "background"`, `placeholder`. 

***

## Media (Composite Uploader)

**What it does:** General media uploader for posts/markers with processing hook.  
**Key options:**

- `type: "media"` in `form`
- Hook `processMedia` runs `onBeforeValidation`. 

***

## Upload File (Generic File)

**What it does:** Single file upload (non-image), with progress, remove, and `saveto` target.  
**Key options:**

- `type: "uploadfile"` (template: `formbuilder_uploadfile`)
- `saveto` (destination key for the uploaded `{ ext,name,path }`), `allowedExtensions`, `module` (uploads to `/upload/{module}/submit`)
- Handles abort/progress/errors and stores file meta in `current[saveto]`.  

***

## Payment Method

**What it does:** Selects a stored payment source (card) and fetches methods from API; persists selection.  
**Key options:**

- `type: "payment"` (template: `formbuilder_payment`)
- Auto-loads `/core/user/bankmethods` and sets default source on first load.  

***

## Preview (Computed / Inline Summary)

**What it does:** Renders a template preview block driven by current form state (e.g., event broadcast counts, promotions).  
**Key options:**

- `type: "preview"` with `form.template` (e.g., `event_broadcast_count`, `event_promotion`); can show/hide dependent sections like `payment` based on API results.  

***

## Button (Single Choice as “Card”)

**What it does:** A single-choice control that maps a `value` to a labeled/described option object; resolves current label/desc from list.  
**Key options:**

- `type: "button"` (template: `formbuilder_button`)
- `options.list[{ value,name,description }]`, `default`.  

***

## Drive (External File/Permission Picker)

**What it does:** Lets users pick an external Drive item, display it, clear it, and change sharing permissions (reader/commenter/writer).  
**Key options:**

- `type: "drive"` (renders item view and a permission menu)
- `saveto` for the selected item; permission update via `/module/drive/setpermissions`.  

***

## Color Picker

**What it does:** Visual color picker with hex input; updates `current[key]`.  
**Key options:**

- `type: "color"` (template uses a `.colorpicker` control). 

***

## URL Name (Slug Helper)

**What it does:** A helper to create a short URL/slug for entities.  
**Key options:**

- `type: "url_name"`. 

***

## PhoneGap / Hidden System Fields

Other frequently used system fields use `hidden` to carry app/session derived data such as IDs, geo indices (`point`), privacy flags, last-active timestamps, default pictures, etc. These are set or transformed by hooks (e.g., `ensureLocation`, `addedBy`) during validation or save.  

***

# Patterns You Can Use Across Elements

- **`default` values**: Many inputs seed `current[key]` with a default if none exists. Redactor/ACE explicitly set empty strings/objects. 
- **Linkage**: Date end can be `linkedTo` start; editors can write to `htmlField`/`textField`; tag pickers can also store `info` side-maps for selected IDs.   
- **Autosave**: Redactor supports debounced autosave with `autosave.versionField`. 
- **Templates**: Most elements accept an override `template` to use a custom `formbuilder_*` template (payment, uploadfile, button, preview, editors).   

***

# Minimal Examples (By Type)

```json
// 1) Tags
"tag": { "type": "tag", "form": { "type": "tags", "endpoint": "[apiurl]/search/tags", "info": "tag_info" } }

// 2) Location
"location": { "type": "object", "fields": { "id": { "type": "string" } },
  "form": { "type": "location", "info": "location.data" } }

// 3) Date (timestamp)
"start": { "type": "timestamp", "form": { "type": "date", "format": "event", "timezoneField": "timezone" } }

// 4) Redactor
"welcome_message": { "type": "text", "form": {
  "type": "redactor", "plugins": ["variable"], "variables": ["[contact.name]"], "nohtml": true } }

// 5) Uploadfile
"brochure": { "type": "text", "form": { "type": "uploadfile", "saveto": "brochure_file" } }

// 6) Buttonselect
"notify_type": { "type": "text", "form": {
  "type": "buttonselect", "default": "hosts", "options": { "order": ["hosts","none"], "list": { "hosts": { "value":"hosts", "name":"Hosts"} } } } }
```

(Examples adapted from schema instances above.)

***

## Notes & Gaps

- I documented every element surfaced in `formbuilder.js` plus every `form.type` used in `schema.json`. If you have custom `formbuilder.templates` entries beyond these, point me to the specific template names and I’ll add them.
- If you want a **matrix** (type → props → template → example), say the word and I’ll generate it from these sources.

If you want this split into separate markdown files per type (or a single “cookbook” page), I can output those next.