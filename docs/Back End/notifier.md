---
title: Notifications (Push / Email / App)
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
# Notifications

The notification system turns application events (“hooks”) into queued messages (email and push). Hooks are defined centrally, enriched with graph data, optionally processed, and then fanned out to per-recipient **notices** that workers deliver at a controlled rate.

***

## Overview (flow)

1. **Emit a hook** – Your app calls `phi::emitHook()` with a hook `id` and minimal `data`. This validates against `system_hooks` and either processes immediately or stores a scheduled job for later processing.\
   *What it does:* creates a normalized “hook” record that anchors all downstream notifications for that event.

2. **Process a hook** – Immediate hooks call `core::processHook(…)` directly; scheduled hooks are picked up from `scheduled_jobs` and passed to the same processor. 

3. **Create notices** – Processing resolves graph references and templates from `system_hooks.json` (e.g., `to`, `from`, `event`, etc.), then writes per-recipient entries into `db.notice` for delivery (email / push). Push inserts use `notice` with `type:'push'`, device and payload metadata, and timestamps.  

4. **Deliver via queues** – A Node worker (`notifier.js`) continuously drains `db.notice` by type with rate limits and back-pressure, handing items to channel-specific senders.   

***

## Emitting hooks

### `phi::emitHook(string $app, int|string $when, array $opts, bool $return=false)`

Validates the hook definition and either processes immediately or schedules a job.\
*What it does:* ensures the hook `id` exists, checks required template keys, and produces a canonical payload the hook processor understands.

* **Required in`$opts`:**

  * `id` – hook key (must exist in `system_hooks`) 
  * `data` – key/value data that must include all keys present in the hook’s `template`. Missing keys are logged and the emit is rejected. 
* **Scheduling vs. immediate:**

  * If `$opts['immediate']` is set, calls `core::processHook(['immediate'=>$save,'qs'=>[]])` synchronously. 
  * Otherwise, behavior depends on `$return`:

    * `$return === false` → **persists** to `scheduled_jobs` (for async processing). 
    * `$return === true` → **returns** the job object without saving (supports bulk insert). Pair with `phi::saveHooks($hooks)`. *Note: current`saveHooks()` implementation appears incomplete; see “Gotchas”.* 

#### Two usage modes (examples)

**Immediate mode (validate + process now + persist results):**

```php
phi::emitHook(phi::$conf['dbname'], time(), [
  'id' => 'affiliate_added',
  'data' => [
    'app_id' => $r['qs']['appid'],
    'to' => ['id' => $v, 'type' => 'user'],
    'from' => phi::keepFields($d['current']['affiliate'], ['id','type']),
    'affiliate_form' => $d['current']['id'],
    'immediate' => 1
  ]
]);
```

**Deferred, bulk mode (generate job records now; save many at once):**

```php
$hooks[] = phi::emitHook(DB, time(), [
  'id' => 'affiliate_added',
  'data' => [
    'app_id' => $r['qs']['appid'],
    'to' => ['id' => $v, 'type' => 'user'],
    'from' => phi::keepFields($d['current']['affiliate'], ['id','type']),
    'affiliate_form' => $d['current']['id']
  ]
], /* return */ 1);

phi::saveHooks($hooks);
```

***

## System hook definitions (`system_hooks.json`)

Each hook id in `system_hooks.json` declares how to enrich (`graph`), pre-process (`process`), template, route, and deliver notifications.

**Common fields** (examples pulled from `event_*`, `comment_*` hooks):

* `id`, `name` – unique key and human label. 
* `required` – channels/config this hook needs (`push`, `email`, `app`, etc.). Delivery won’t run if requirements aren’t met. 
* `modulePath` – base folder for templates. 
* `variables` – available template tokens like `[from.data.name]`, `[event.id]`. 
* `route` – link path substituted into templates (e.g., `/event/[event.id]`). 
* `tofield`, `to_id_field`, `to_email_field` – where resolved “to” data lives and which fields represent id/email. 
* `graph` – rules to fetch related documents and inject them into the payload before templating (e.g., resolve `to.id` to `to.data` from `user`/`page`; fetch `event`, `comment`). Supports `coll`, `to`, `match`, `opts`, `clearOnNull`.  
* `graph2` – secondary graph pass for deeper joins (e.g., resolve `receipt.event_time` to `receipt.event_time_info`). 
* `template` – minimal required keys (and default shapes) that **must** be present in `emitHook(..., ['data'=>…])`. 
* `process` – optional transforms to run pre-template (e.g., `event_pretty_time`, `addspaces` for fields, `processPictures` resizing, `checkOverwriteSettings` by `event.id`).  
* `attach` – named attachments to include (e.g., `["event_tickets","event_invite"]`). 
* Email-specific flags: `email_template`, `email_template_full`, `event_default_reply_to`. 

> **Example: Time change to ticket holders**\
> Requires `push`, `app`, and `email`; graphs `to`, `event`, and `receipt`; processes pretty time, spacing, and pictures; attaches ticket PDFs; and routes to `/event/[event.id]`.    

***

## Delivery workers & queues (`notifier.js`)

A dedicated worker drains `db.notice` per channel with adjustable throughput and queue size caps.

* **Init & limits** – `enabled:['email','push']`, `limit.email.send_rate`, `limit.email.queue_size`, and push equivalents; polls every `delay` ms.  
* **Fetch windows** – Uses `uts` (microtime) to fetch new notices since the last processed value and excludes `status:-1`. 
* **Email queue** – `async.queue(..., send_rate)` processes up to `send_rate` concurrent sends, then logs drain. 
* **Push queue** – Similar pattern; items with `to`/device metadata go through `push.send`.  
* **Update status** – After send, `db.notice.updateOne({_id}, update)` with write concern note in comments. 

*What it does:* provides back-pressure and elasticity—keeps delivery smooth during spikes and prevents DB hot spots by paging on `uts`.

***

## Push notice structure (example saver)

When queuing push, the server writes a `notice` document shaped like:

```js
{
  type: 'push',
  to: <user_id>,
  opts: {
    notId: <10-char guid>,
    message, title, messagedata, intent, sound, count,
    device: { arn, sandbox, type, version }
  },
  ts: <unix>,
  uts: <microtime>
}
```

This is exactly how your PHP push helper persists the notice before workers pick it up. 

***

## Defining new hooks (checklist)

1. **Add a hook object** to `system_hooks.json`: choose `id`, `required`, `graph`, `process`, `variables`, `route`, and `template` keys. Use existing `event_*` / `comment_*` hooks as patterns.  
2. **Emit** with `phi::emitHook($app, $when, ['id'=>..., 'data'=>...])`. Ensure **all** `template` keys are present in `data`. 
3. For bulk workflows, pass `$return=1` and call `phi::saveHooks($hooks)` once. 
4. If you must run immediately (synchronous path), include `'immediate'=>1` in `data`. 

***

## Gotchas & considerations

* **Strict template validation.** `emitHook` logs and aborts if your `data` is missing any key found in the hook’s `template`. Add placeholders or adjust the template for optional fields (or use `clearOnNull` in your `graph` where appropriate).  
* **Bulk save helper looks incomplete.** `phi::saveHooks()` currently pushes `$hooks` inside the loop instead of `$v`, which would duplicate the whole array; confirm/patch before relying on bulk mode in production. 
* **Queue pressure.** Default `send_rate` and `queue_size` are both `10`/`500` per channel. Tune for SES/SNS limits and mailbox provider guidance to avoid throttling or spam classification. 
* **De-dupe & idempotency.** Workers page by `uts`; if you reinsert the same notice with an earlier `uts`, it won’t be re-read. Prefer monotonic `uts` and guard against duplicate emits upstream. 
* **Immediate mode = synchronous work.** Using `'immediate'` shifts work onto the request path (via `core::processHook`). Reserve for small fan-outs or administrative operations; use scheduled jobs for bulk events. 

***

## Quick reference

* **Emit now:** set `'immediate'=>1` in `data`. 
* **Batch emit:** call with `$return=1`, then `phi::saveHooks($hooks)`. 
* **Define data shape:** add/verify `template` & `graph` in `system_hooks.json`.  
* **Delivery engine:** `notifier.js` drains `db.notice` by type with rate limits.  

If you’d like, I can also add a short “How to add a new hook end-to-end” tutorial with a minimal `system_hooks.json` entry + emit + end-to-end test.

Medical References:

1. None — DOI: file-NCTTQmkzwMHQzahw6vzdDt
2. None — DOI: file-5FrXVsjx5o6Uiv9Ntogfub
3. None — DOI: file-H3epvt3KSJTStuxP88sn4i
