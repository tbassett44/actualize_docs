---
title: Infinite Scrolling
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
# Module: `modules.infinitescroll` — Infinite/virtual list with paging, sockets, search & filters

## High-level summary

A flexible scroller that renders list pages from either an **API endpoint** or an **in-memory dataset**, auto-loads more when you near a “waypoint,” and optionally wires **live updates** (sockets), **search**, **filters**, **horizontal carousels**, **sticky headers**, and a **reverse/chat mode** that maintains scroll position as new items arrive. It’s built around a small state machine (loaded/last/mostRecent), a render pipeline (`phi.render` + templates), and a paging contract (`data.order` + `data.list`). 

***

## Quick start

```js
const feed = new modules.infinitescroll({
  ele: $('#feed'),                        // container for list
  endpoint: app.apiurl + '/posts/list',   // server returns { success, data:{ order, list, last, extra? } }
  opts: { max: 15, filter: {} },          // sent to endpoint (merged per request)
  template: 'post_item',                  // item template used by phi.render
  loaderClass: 'my-loader',               // optional: style of initial loader
  onPageReady(ele, pageData) { /* bind per-page events */ }
});
```

When the hidden `.waypoint` enters view, the next page loads automatically; an “End of List” row appears when no more items are available. 

***

## Options (most useful)

| Option                                                                               | Type                                                        |  Default | What it does                                                                                                    |
| ------------------------------------------------------------------------------------ | ----------------------------------------------------------- | -------: | --------------------------------------------------------------------------------------------------------------- |
| `ele`                                                                                | jQuery                                                      |        — | List element the module renders into.                                                                           |
| `endpoint`                                                                           | string                                                      |        — | API URL; response should include `data.order[]`, `data.list{}`, `data.last`.                                    |
| `data`                                                                               | object/false                                                |        — | Local dataset alternative; module slices from it when `endpoint` is not provided.                               |
| `opts`                                                                               | object                                                      |     `{}` | Sent with each request (`search`, `filter`, etc.). `max` can be set per request.                                |
| `max`                                                                                | number                                                      |     `15` | Default page size; also used to detect “has next” in some flows.                                                |
| `template`                                                                           | string                                                      |        — | Item template. When `viewselect` is active, it can override via `getTemplate()`.                                |
| `renderData`                                                                         | any                                                         |        — | Extra data passed into item templates.                                                                          |
| `horizontal`                                                                         | bool                                                        |  `false` | Enables horizontal carousel mode (snap optional). Uses different scroll logic.                                  |
| `snapTo`                                                                             | number/string                                               |        — | Snap points in horizontal mode.                                                                                 |
| `reverse`                                                                            | bool                                                        |  `false` | Chat-style: prepend new pages at top, preserve visual position on load.                                         |
| `search`                                                                             | \{input, closer, endpoint?, allowBlank?, change?, onClear?} |        — | Wires an input for live search and reloads on change; can use a **separate search endpoint**.                   |
| `filter` / `filter2`                                                                 | object                                                      |        — | Integrates with `modules.filter`; updates `opts.filter` then reloads. (`filter2` can trigger initial `load()`.) |
| `viewselect`                                                                         | object                                                      |        — | Integrates with `modules.viewselect`; toggles `inline` layout and reloads on change.                            |
| `channel`                                                                            | string                                                      |        — | If set, subscribes to socket updates and applies `onUpdate/onCreate/onDelete`.                                  |
| `onUpdate`                                                                           | fn                                                          |        — | Called after any socket update (passes the updated item keyed by `getDataKey()`).                               |
| `datakey` / `lastKey` / `recentKey`                                                  | string                                                      | see code | Keys to identify items, track “last,” and compute `mostRecent`. Auto-fallback to `_id` if field missing.        |
| `header`                                                                             | \{rule, template}                                           |        — | Enables sticky section headers (e.g., by date via `rule: 'samedate'`). Uses Waypoints.                          |
| `sticky`, `mainStickyElements`, `stickyEle`, `stickeyOffset`                         | selectors/num                                               |        — | Adds scrolling sticky clones into a top slot; offset adapts to phone notch.                                     |
| `buttonLoad`                                                                         | bool                                                        |  `false` | Show a **Load More** button instead of auto-load when waypoint hits.                                            |
| `checkNextPage`                                                                      | bool                                                        |        — | Only auto-fetch next if the last page looked “full” (or lastCount != 0).                                        |
| `disableNextPage`                                                                    | bool                                                        |        — | Turns off pagination.                                                                                           |
| `endOfList`, `noResults`, `endOfListColor`                                           | string                                                      | see code | Footer copy/colors when done or empty.                                                                          |
| `onLoad`, `onFirstLoad`, `onLoadError`, `onExtraData`, `onAsyncReady`, `onPageReady` | fns                                                         |        — | Hooks for various stages in the load/render lifecycle.                                                          |
| `disableResume`, `clearOnResume`, `disableUpdateOnStart`, `reloadele`                | flags                                                       |        — | Control auto-refreshing and the “new content available” banner logic.                                           |
| `dataType`, `timeout`, `processResponse`                                             | —                                                           |        — | Fine-tune API request/response handling. Uses `modules.api` under the hood.                                     |

***

## Public methods (instance)

* **`reload(cb?)`** – Clears list state & UI, shows loader, and fetches the first page again. Resets `last`, `order`, `mostRecent`. 
* **`load(cb?, newest?, dontRender?)`** – Core loader; builds request params (`last`/`after`), calls API or slices local data. When `newest` is set, checks for newer content without moving paging cursor. 
* **`add(item, update?)`/`remove(item)`** – Insert or update an item (or delete it) in place; supports socket “update/create/delete” paths. 
* **`getById(id)`/`getIndexById(id)` /`getList()`** – Read helpers over the internal `{order,list}` store. 
* **`setData(data)`** – Replace in-memory dataset then `reload()`. 
* **`scrollTop()`** – Smoothly scroll container to top. 
* **`enable()`/`disable()`** – Toggle the underlying `modules.scroller`. 
* **`start()`/`stop()`** – Begin/stop interval checks & socket subscription; also used by visibility/resume flows. 
* **`onResume(cb?)`** – On app resume: either reload everything or try to fetch just the newest page; can invoke `cb(mostRecent)` if something new arrived. 
* **`updateHeight()`** – Calls `Waypoint.refreshAll()` when container size changes. 

***

## Data contract (expected response)

Server responses are normalized as:

```json
{
  "success": true,
  "data": {
    "order": ["postId3", "postId2", "postId1"],
    "list": {
      "postId3": { /* item */ },
      "postId2": { /* item */ },
      "postId1": { /* item */ }
    },
    "last": 30,
    "extra": { /* optional extras for header rows / summary */ }
  }
}
```

* `order` drives the render order; `list` is the keyed dictionary.
* `last` is fed back into subsequent requests as the paging cursor.
* `extra` is passed to `onExtraData()` for custom header/summary rendering. 

***

## Rendering & templates

* The list frame uses `module_infinitescroll_page` (or `infinitescroll_inline` when `inline`/`viewselect` is active). Items render through your `template` or `getTemplate()` override. A `.waypoint` sentinel element is managed automatically. Endcap rows show **End of List** / **No Results**. 
* **Horizontal mode** computes container widths, sets fixed height, and places the waypoint at the tail; optional `snapTo` enables snapping. 
* **Sticky headers** use Waypoints and clone the last seen sticky element into a top slot (`infinitescroll_sticky`). iOS notch offsets are handled. 

***

## Search & filters

* **Search:** Bind an input that writes to `opts.search` on each keystroke and calls `reload()`. Optional closer icon clears and hides when blank; a **separate search endpoint** can be supplied. 
* **Filters:** Plug `modules.filter` via `filter`/`filter2`. They write `opts.filter` and trigger `reload()` or initial `load()`. `setFilterItem(id,val)` can programmatically update an active filter. 

***

## Live updates (sockets / listen)

* If `channel` is set, the scroller subscribes to `app.user.startSocket(channel, onUpdate)` and handles `onCreate`, `onUpdate`, `onDelete`. Missing data leads to `reload()`. 
* With `listen` and `context`, it calls `phi.listen(context, id, onEmitData)` per item to react to granular updates. 

***

## Error handling & retry

* API/network failures call `onLoadError(resp)` if provided; otherwise a default **error row** is rendered with a **Retry** button that calls `load()` again (unless disabled for specific error codes via `noRetry`). 

***

## Reverse/chat mode details

When `reverse: true`, new content can be appended **visually at the top** while preserving the reader’s position:

* On page render, the module captures current container height & scroll offset, then **scrolls back** to maintain visual position after DOM growth.
* `onResume()` can fetch only “newest” items (using `after=mostRecent`) and prepend them. 

***

## Horizontal carousel notes

* Uses `modules.scroller` with `scrollX: true`.
* Waypoint detection is based on **percent width remaining** rather than vertical offsets.
* Optional **“Load More” button** (`buttonLoad: true`) is supported if you prefer tap-to-fetch. 

***

## Practical examples

### 1. Classic feed with search & filters

```js
const feed = new modules.infinitescroll({
  ele: $('#feed'),
  endpoint: app.apiurl + '/events/list',
  opts: { max: 20, filter: {} },
  template: 'event_item',
  search: {
    input: $('#q'),
    closer: $('.clear-search'),
    endpoint: app.apiurl + '/events/search'
  },
  filter: { /* modules.filter options */ },
  onPageReady(ele, page) { /* bind per page */ }
});
```

### 2. Chat timeline (reverse mode) with socket channel

```js
const chat = new modules.infinitescroll({
  ele: $('.chat-list'),
  endpoint: app.apiurl + '/chat/list',
  template: 'chat_message',
  reverse: true,
  channel: 'room:123',
  onPageReady(ele) { /* autolinker, timestamps, etc. */ }
});
```

### 3. Horizontal carousel with button-to-load

```js
new modules.infinitescroll({
  ele: $('.cards'),
  horizontal: true,
  height: 260,
  template: 'card_item',
  endpoint: app.apiurl + '/cards/list',
  buttonLoad: true,
  offset: '200%'  // start loading when within 2 viewport widths
});
```

***

## Things to watch out for

* **Response shape:** If your endpoint doesn’t return `{order,list,last}`, adapt it via `processResponse(resp)` to that shape. 
* **`datakey`fallback:** If your objects don’t carry the id field you set, the module falls back to `'_id'`. Ensure consistency to avoid duplicate renders. 
* **Waypoints & container resize:** After changing heights (e.g., images load), call `updateHeight()` to refresh Waypoint positions. 
* **Mobile keyboard interactions:** Optional “swipe-to-close keyboard” behavior on iOS/PhoneGap can be enabled via `swipeToClose`; it hides the keyboard when the user scrolls upward quickly.
