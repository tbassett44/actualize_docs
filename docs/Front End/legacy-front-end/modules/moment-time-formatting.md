---
title: Moment / Time Formatting
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
[moment.js Documentation](https://momentjs.com/) 

# `modules.moment` — date/time utilities (Moment + TZ)

## Overview

A lightweight wrapper around **moment** and **moment-timezone** that normalizes timestamps from multiple sources (plain JS dates, PHP/Unix, MongoDB ObjectIds and `$date` payloads), formats them for UI, calculates diffs/ages, and handles cross-timezone rendering with optional event “end” times and relative labels (e.g., “Happening Now”, “Yesterday”, “in 3 days”). It also bundles a current timezone dataset for browser use.

## Dependencies & globals

- `moment` and `moment.tz` (moment-timezone is bundled at the bottom of the file)
- `_.getTimeZone()` (your project’s helper that returns the local IANA zone)
- `app.time` (optional “current time” override for deterministic rendering)
- Exposes under `window.modules.moment`

## API

### `months`

Map `0..11 → { short: 'Jan' | 'Feb' | ... }`. Use for static labels.

***

### `getTimezoneAbr(zone: string): string`

Returns the timezone **abbreviation** for an IANA zone, e.g., `America/Denver → MDT` (or MST depending on date). Internally: `moment().tz(zone).zoneAbbr()`.

***

### `getTs(ts: any, php?: boolean): number|false`

**Normalizes** a variety of timestamp inputs to a **JavaScript epoch milliseconds** number.

Accepts:

- Plain JS `Date` or parseable date string
- **PHP/Unix seconds** when `php===true` → multiplied by 1000
- MongoDB:

  - `{"$oid": "…"}:` decodes ObjectId’s first 4 bytes (seconds), _minus 30s_ (heuristic cushion)
  - ISO string ending in `.000Z` → `new Date(...).getTime()`
  - `{"$date": <number|string|{"$numberLong": "…" } >}` → number
- `{ day, month }` → constructs a date at **12:00** local on that month/day this year

Returns `false` if parsing fails and logs a console warning.

***

### `parse(str: string): number`

Thin wrapper over `Date.parse(str)`, returns **Unix seconds** (floored). Useful for rough parsing.

***

### `getAge(ts: any, php?: boolean): number|string|false`

Age in **years** from now to `ts`. Returns `''` if no `ts`, `false` if invalid.

***

### `getDiff(ts: any, php?: boolean, type: moment.unitOfTime.Diff): number|string|false`

Generalized difference from **now** to `ts` in the given unit (`'days'`, `'hours'`, …). Same return semantics as `getAge`.

***

### `getTimezoneDiff(zone1: string, zone2: string): number`

Difference between two zones’ **UTC offsets at the current moment**, returned in **seconds**. Positive means `zone1` is ahead of `zone2`.

***

### `format(ts: any, type?: string, end?: any, php?: boolean, timezone?: string): string|false`

The workhorse formatter. Normalizes `ts` (and optional `end`) via `getTs`, then renders a string based on `type`. If `timezone` is provided, it formats using that IANA zone (mitigating DST shifts); otherwise it uses `_.getTimeZone()`.

Common `type` options (selected highlights):

- **`simplerelative`** – `m.fromNow()`
- **`ago`** – Relative (“2 hours ago”). If both `end` and `app.time` are present, shows **“Happening Now”** when `app.time` is between `ts` and `end`.
- **`ago_day`** – Human day buckets: `Today`, `Yesterday`, `Tomorrow`, `in N days`, `N days/month(s) ago`
- **`chat_ago` / `chat`** – Compact chat timestamp logic (time today, weekday if \<4 days, or `MMM Do`)
- **`date` / `prettydate` / `nicedate`** – Various fixed patterns (`l`, `ddd MMM Do`, `dddd MMMM Do`)
- **`ts`** – Returns **ms epoch** as number via `moment(ts).format('x')`
- **Event display**:

  - `timerange`, `times`, `event_time` – `h:mm a` ranges
  - `event`, `event_full`, `event_full_short`, `eventheader`, `prettyevent` – verbose strings with day/month and, when provided, an end time; some variants append a timezone **abbr**
- **Calendar helpers**:

  - `start_of_day` → start-of-day epoch (ms)
  - `calendar_lastdate` → `YYYY-MM-DD`
  - `calendar_time` → `HH:mm`
- **`simpledate`** – Returns `Today` / `Yesterday` or `M/D/YY` (respects `timezone` if provided)
- **`time`** – `h:mm a` or a range with `end`

Returns `''` for empty input, `false` for invalid timestamps.

> Note: Many variants also append the **event year** when it differs from the current year (e.g., `" (2024)"`).

***

## Usage examples

```js
const m = modules.moment;

// 1) Normalize mixed timestamps
const ts1 = m.getTs('2025-10-03T12:00:00Z');        // ms
const ts2 = m.getTs(1696262400, true);              // PHP seconds → ms
const ts3 = m.getTs({ "$oid": "651c6f8d0000000000000000" }); // ObjectId heuristic

// 2) Relative chat labels
m.format(ts1, 'chat_ago');                          // e.g., "Fri"
m.format(ts1, 'chat');                              // "1:23 pm (2 days ago)"

// 3) Cross-timezone event with end time
m.format(ts1, 'event_full', ts1 + 90*60*1000, false, 'America/Denver');
// "Friday, Oct 3 at 12:00 pm - 1:30 pm MDT (2025)"

// 4) Age and differences
m.getAge('1989-06-24');                             // 36
m.getDiff(Date.now() + 3*864e5, false, 'days');     // -3 (future)
```

## Design notes & edge cases (recommended fixes / cautions)

- **Duplicate switch case**: `case 'birthday'` appears twice with different formats (`'MMM Do'` and `'MMM Do, YYYY'`). Pick one identifier (e.g., keep `'birthday'` for month/day; rename the full one to `'birthday_full'`) to avoid dead code paths.
- **Undeclared variables in `'event'`**: Uses `event_year` / `current_year` before they’re defined—define them early (like in `eventheader`) to avoid `ReferenceError`.
- **Timezone abbreviation**: Several branches append `moment.tz.zone(timezone).abbr(360)`. `zone.abbr()` expects an **epoch ms** to resolve DST-aware abbr, not `360`. Pass the event instant (e.g., `m.valueOf()`) to ensure correct DST labels.
- **DST adjustments commented out**: There’s commented logic for manual DST correction. Since you call `moment(ts).tz(timezone)`, Moment-TZ already handles DST. Keeping the manual code disabled is correct to avoid double-shifting.
- **`app.time` test path**: When set, it’s converted with `getTs(app.time, 1)` (treating as PHP seconds). Ensure `app.time` really is seconds; otherwise you’ll shift by 1000×.
- **ObjectId → “minus 30s” heuristic**: `getTs` subtracts 30 seconds from ObjectId-derived seconds. It may be intentional (ordering cushion) but document it so readers aren’t surprised by slightly “earlier” times.
- **`parse()` returns seconds**: Everything else returns ms; call-sites must not mix units. Prefer `getTs` unless you specifically need seconds.

## When to prefer each formatter

- **Lists, feeds, chat** → `simplerelative`, `comment`, `chat_ago`
- **Event cards** → `event_full_short` (short day + clear range + tz abbr)
- **Detail pages** → `event_full` (includes day name and year if different)
- **Dashboards** → `prettydate` / `nicedate` for consistent visual rhythm

***