---
title: Geocoding
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
# Module: `modules.geocode` (Mapbox Geocoding helper)

## High-Level Summary

A small wrapper around the **Mapbox Geocoding API** that normalizes inputs/outputs for your app, adds optional **type filtering** and **location proximity**, and provides helper utilities for parsing text and coordinates. It rate-limits overlapping calls (soft lock + queued retry) and returns **raw Mapbox `features`** on success.  
See Mapbox docs: [Mapbox Geocoding API](https://docs.mapbox.com/api/search/geocoding/).

***

## Options / Props

> The module is a singleton with fixed configuration and a single public search method.

| Name                    | Type       | Default                         | Description                                                                                                                 |                                                                             |
| ----------------------- | ---------- | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `key`                   | `string`   | Mapbox **public** token in code | Access token appended to each request. **Security note:** consider externalizing.                                           |                                                                             |
| `type`                  | `string`   | `"mapbox.places"`               | Dataset used in the request path.                                                                                           |                                                                             |
| `init()`                | `function` | —                               | Calls `self.location.start()` (assumes a global location module).                                                           |                                                                             |
| `getTypes(opts)`        | `function` | —                               | Translates `opts.types` to Mapbox `types` param. Built-in aliases: `"(cities)" → "place"`, `"(pois)" → "poi"`.              |                                                                             |
| `getText(feature)`      | `function` | —                               | Extracts human-friendly bits from a Mapbox `feature` into `{ text, place_name, ...context }`.                               |                                                                             |
| `parseData(resp)`       | `function` | —                               | Returns `resp.features` (stored in `self.cdata`).                                                                           |                                                                             |
| `getLoc(data, cb?)`     | `function` | —                               | Normalizes coordinates to `{ lng, lat }` from either a Mapbox `feature` or a `{ coordinates:{longitude,latitude} }` object. |                                                                             |
| `search(val, opts, cb)` | `function` | —                               | Core method. Issues request and calls \`cb(features                                                                         | false)\` with parsed results. Queues latest term if a request is in flight. |

***

## `search(val, opts, cb)` Details

**Signature:** `modules.geocode.search(queryString, options, callback)`

| Option           | Type       | Default         | Effect                                                                                                                            |
| ---------------- | ---------- | --------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `opts.types`     | `string[]` | `[]`            | Filter Mapbox results by type. Accepts Mapbox types (`"place"`, `"poi"`, `"address"`, …) and aliases `"(cities)"` and `"(pois)"`. |
| **Proximity**    | —          | device location | If `phone.location.getNearestLocation()` returns `{lng,lat}`, appends `proximity=lng,lat` to bias results.                        |
| **Autocomplete** | —          | `true`          | Always set. Enables prefix matching as user types.                                                                                |

**Behavior:**

- Debounce-ish queue: if `search` is called while `processing` is true, it **stores the latest call** and replays it after the current request completes (with a 500ms safety timer).
- Builds endpoint:  
  `https://api.mapbox.com/geocoding/v5/{type}/{encoded query}.json?autocomplete=true&types=...&access_token=...&proximity=...`
- Uses `modules.api` with `{ type:'GET', cleanData:true, dataType:'json' }` (no auth/query pollution).
- Success: if `data.features.length`, calls `cb(features)`; else `cb(false)`.

***

## Helpers

### `getTypes(opts)`

- Input: `{ types?: string[] }`
- Output: `string[]` of Mapbox types
- Mapping:

  - `"(cities)"` → `"place"`
  - `"(pois)"` → `"poi"`
  - Unknown entries pass through unchanged.

### `getText(feature)`

- Returns a compact object for display/search highlights:

  - `text` (short label)
  - `place_name` (full label)
  - Context parts keyed by their type, e.g. `{ country: "United States", region: "Colorado" }` based on `feature.context[].id`.

### `getLoc(data, cb?)`

- Accepts:

  - Mapbox `feature` (`feature.geometry.coordinates = [lng, lat]`)
  - FB-style object: `{ coordinates:{ longitude, latitude } }`
- Returns `{ lng:Number, lat:Number }`. If `cb` provided, calls and returns that value.

***

## Usage Examples

### 1. Cities only, with UI update

```js
modules.geocode.search('bould', { types: ['(cities)'] }, function(features){
  if (!features) return showEmpty();
  const items = features.map(f => ({
    label: modules.geocode.getText(f).place_name,
    loc: modules.geocode.getLoc(f)
  }));
  renderList(items);
});
```

### 2. Mixed: places + POIs, extract lat/lng

```js
modules.geocode.search('coffee near union', { types: ['(cities)', '(pois)'] }, function(results){
  if (!results) return;
  const first = results[0];
  const { lng, lat } = modules.geocode.getLoc(first);
  centerMap({ lng, lat });
});
```

### 3. Use raw features for advanced UI

```js
modules.geocode.search('denver', {}, (features) => {
  // features are raw Mapbox features; keep for later selection
  state.searchResults = features;
});
```

***

## Returned Data (Mapbox Feature Quick Reminders)

A typical feature includes:

- `place_name`: full human label (`"Denver, Colorado, United States"`)
- `text`: primary token (`"Denver"`)
- `center` / `geometry.coordinates`: `[lng, lat]`
- `place_type`: e.g., `["place"]`, `["poi"]`
- `context`: array of parent objects with `id` like `country.123`, `region.456`, etc.

Your helpers (`getText`, `getLoc`) already normalize the most commonly needed pieces.

***

## Notes & Considerations (poking holes)

1. **Hard-coded access token**  
   Token is embedded in source (`modules.geocode.key`). Move to a **server-side config**, environment variable, or injected at build time to reduce exposure and ease rotation.

2. **Minimal error handling**  
   Failures return `cb(false)` silently. Consider surfacing network errors/timeouts via an optional `onError` or by passing `{ error }` objects.

3. **Throttling vs. Debounce**  
   Current “latest wins” queue helps, but for aggressive typing a **true debounce** (e.g., 200–300ms) could further reduce requests and Mapbox costs.

4. **Type coverage**  
   Only two friendly aliases are baked in. If you need addresses or neighborhoods, pass Mapbox types directly (`'address'`, `'neighborhood'`, `'locality'`, `'district'`, `'postcode'`, etc.).

5. **Proximity bias**  
   Relies on `phone.location.getNearestLocation()` being available and synchronous. If not present or stale, results might be less relevant. Consider fetching async and retrying with proximity once available.

6. **Result shaping**  
   `parseData` returns raw features. If the UI expects a normalized array (id, title, subtitle, coords), you might add a small mapper to avoid duplicating logic in callers.

7. **Rate limits / quotas**  
   Mapbox rate limits apply. Add caching of recent queries and/or a minimum query length (`val.length ≥ 2`) to cut noise.