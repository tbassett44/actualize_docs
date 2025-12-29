---
title: Video Embedding
deprecated: false
hidden: false
metadata:
  robots: index
---
This document describes how the Actualize video embedding flow works end-to-end:

* The **frontend embed module** (`modules.video_embed`)
* The **iframe player endpoint** (`player.php`)
* The **postMessage bridge** between parent and iframe
* The **query-string (QS) parameters** supported by the player
* The **analytics / watch-time tracking** behavior

Based on `video_embed.js` and `player.php`.

***

## 1. High-level architecture

1. The calling page instantiates `modules.video_embed(options)`.
2. `video_embed` builds an iframe URL pointing at `app.playerurl + '/' + type + '/' + id`, and appends QS params (`unique_id`, `token`, `nocontrols`).
3. The iframe loads `player.php`, which:

   * Authenticates the user (via `API::authUser`).
   * Selects the correct video provider (`youtube`, `vimeo`, or `video`).
   * Renders a Plyr.js instance in the iframe.
4. The iframe and parent communicate via `window.postMessage`:

   * The **player** sends events up (e.g., `timeupdate`, `loadedmetadata`, `pause`, `ended`).
   * The **parent** sends control actions down (e.g., `play`, `pause`, `seek`, `fullscreen`).
5. The player reports watch progress and key events to `/v1/schema/video_view` for analytics.

***

## 2. Frontend: `modules.video_embed`

### 2.1. Constructor options

```js
modules.video_embed({
  renderTo: HTMLElement,       // Required: container element for the iframe
  type: 'youtube' | 'video',   // Player type (see below)
  id: 'pK_axPtr23Y',           // Video identifier (YouTube ID, DB id, etc.)
  controls: true | false,      // Optional, defaults to true
  nobigplay:true | false,				// Optional, defaults to false
	events: {                    // Optional callbacks
    ready: function(){},
    loadedmetadata: function(meta){},
    timeupdate: function(timeInfo){}
  }
});
```

**Supported `type` values:**

* `youtube` – YouTube video embedding.
* `vimeo` – Vimeo embedding (supported in `player.php`, even though `video_embed.js` comments list YouTube/video).
* `video` – Direct MP4 video handled by your own backend / DB.

### 2.2. What `modules.video_embed` does

On `init()`:

1. Generates a `unique_id` for this player instance: `Math.uuid(8)`.

2. Constructs the base URL:

   ```js
   var url = app.playerurl + '/' + options.type + '/' + options.id;
   ```

3. Builds a query-string object `qs`:

   * `unique_id` – random per-iframe instance.
   * `token` – from `app.user.profile._id` if available, otherwise a hardcoded test token.
   * `nocontrols=1` – if `options.controls === false`.

4. Serializes `qs` into `?key=value&...` and appends to `url`.

5. Creates the `<iframe>` element:

   ```js
   var iframe = document.createElement('iframe');
   iframe.src  = url;
   iframe.id   = self.unique_id;
   iframe.style = 'width:100%;height:100%;border:0px;';
   iframe.allowFullscreen = true;
   options.renderTo.appendChild(iframe);
   ```

6. Registers a `window.addEventListener('message', self.handleMessage)` listener to receive messages from the iframe.

7. Exposes a set of control methods on the instance:

   ```js
   this.play();
   this.pause();
   this.stop();
   this.restart();
   this.seek(timeInSeconds);
   this.enterFullScreen();
   this.exitFullScreen();
   this.toggleFullScreen();
   this.destroy();
   ```

   All of these call `self.sendMessage({ ... })`, which posts messages into the iframe.

***

## 3. Player endpoint: `player.php`

### 3.1. Route and authentication

`player.php` expects routes of the form:

```text
https://player.actualize.earth/{provider}/{videoId}?unique_id=...&token=...&nocontrols=1
```

On load:

1. Reads configuration.

2. Parses the request via `API::parseRequest()`.

3. Authenticates user via `API::authUser($r)`.

4. If the user is not authenticated (`!isset($r['auth']['uid'])`), it returns a `401` with message:

   > You must be Logged in to view this

5. Requires `unique_id` in the query string. If missing:

   > 404: `unique_id` parameter must be set

So in practice:

* **`token`** (or some equivalent auth mechanism) must resolve to a valid user.
* **`unique_id`** must always be included (the frontend guarantees this).

### 3.2. Provider handling

`player.php` switches on `$r['path'][1]`:

* `vimeo`

  * `$vid = $r['path'][2]` (required).
  * Renders a `<div class="plyr__video-embed" id="player">` with a Vimeo `<iframe>` inside.

* `youtube`

  * `$vid = $r['path'][2]` (required).
  * Renders a similar Plyr embed wrapper with a YouTube `<iframe>`.

* `video`

  * `$r['path'][2]` is the internal video id:

    * If id is `'test'`, uses a hard-coded S3 demo MP4.
    * Else, loads the video from `db2::findOne(DB, 'video', ['id' => $r['path'][2]])`.
  * Uses a `<video id="player" class="plyr__video-embed">` tag with `src` and `poster` configured.

If the provider or id is invalid, the player responds with `404: Video Not Found`.

### 3.3. Controls configuration

At the PHP level:

```php
$controls = [
  'play',
  'progress',
  'current-time',
  'mute',
  'volume',
  'fullscreen'
];

if (isset($r['qs']['nocontrols'])) {
  $controls = [];
}
```

These are passed into Plyr:

```js
app.player = new Plyr('#player', {
  autoplay: false,
  controls: <?php echo json_encode($controls); ?>,
  fullscreen: { enabled: true, iosNative: false, fallback: true }
});
```

So:

* By default, a standard control set is shown.
* If the `nocontrols` QS param is present (any value), all controls are removed.

***

## 4. Query-string parameters

These are the main QS parameters that should be documented.

### 4.1. `unique_id` (required)

* **Type:** string
* **Set by:** `modules.video_embed` (random `Math.uuid(8)`)
* **Used by:**

  * `player.php` – required; if missing, request fails.
  * The iframe’s JS – included in each `postMessage` to the parent.
  * The parent (`video_embed.js`) – filters incoming messages so only events for that specific iframe are processed.

**Purpose:**
Identifies a particular iframe/player instance and ensures messages are routed to the correct `modules.video_embed` instance.

***

### 4.2. `token` (required for auth)

* **Type:** string (user auth token / profile id)
* **Set by:** `modules.video_embed`, taking from `app.user.profile._id` if available; otherwise a test token.
* **Used by:** The backend auth layer (`API::authUser`), not directly by `player.php` logic.

**Purpose:**
Authenticates the request so only logged-in users with a valid token can view and interact with the video.

***

### 4.3. `nocontrols`

* **Type:** boolean flag (presence-based)
* **Set by:** `modules.video_embed` when `options.controls === false`.
* **Used by:** PHP:

  ```php
  if (isset($r['qs']['nocontrols'])) {
    $controls = [];
  }
  ```

**Purpose:**
Allows the embedding context to hide all onscreen player controls (useful for custom UI overlays or kiosk modes). The player can still be controlled via the postMessage API from the parent page.

<br />

### 4.4. `nobigplay`

* **Type:** boolean flag (presence-based)
* **Set by:** `modules.video_embed` option `options.nobigplay`.
* **Used by:** PHP:

  ```php
  $controls=[
        'play',
        'progress',
        'current-time',
        'mute',
        'volume',
        'fullscreen'
      ];
      if(isset($r['qs']['nocontrols'])){
        $controls=[];
      }
      if(isset($r['qs']['nobigplay'])&&$r['qs']['nobigplay']=="1"){
      }else{
        $controls[]='play-large';
      }
  ```

**Purpose:**
Allows for turning on or off the big play button at the center of the video player

***

## 5. Message bridge: parent ↔ iframe

### 5.1. Message format from parent → iframe

`modules.video_embed` sends messages using:

```js
this.sendMessage = function (action) {
  action._provider = 'actualize_player';
  var frame = document.getElementById(self.unique_id);
  if (frame && frame.contentWindow) {
    frame.contentWindow.postMessage(action, "*");
  }
};
```

Actions used:

* `play`
* `pause`
* `stop`
* `restart`
* `seek` (with `time: <seconds>`)
* `fullscreen` (with `mode: 'enter' | 'exit' | 'toggle'`)

Example:

```js
playerInstance.seek(120);         // Sends { action: 'seek', time: 120, _provider: 'actualize_player' }
playerInstance.toggleFullScreen() // Sends { action: 'fullscreen', mode: 'toggle', _provider: 'actualize_player' }
```

In the iframe (`player.php` JS):

* `window.addEventListener('message', app.onMessage, false)` is registered when inside an iframe.
* `app.onMessage(e)`:

  * If `e.action` exists, it’s treated as an internal call.

  * Otherwise, it validates:

    ```js
    if (!e.data || !e.data._provider || e.data._provider != app.provider) return false;
    e = e.data;
    ```

  * Switches on `e.action`:

    * `get_info` – emits an `info` event back to the parent.
    * `set_info` – reserved for future use.
    * `seek` – sets `app.player.currentTime = e.time`.
    * `fullscreen` – calls `app.player.fullscreen.enter/exit/toggle`.
    * Any other supported method name (e.g., `play`, `pause`, `stop`, `restart`, `rewind`, etc.) is called directly on the Plyr instance.

***

### 5.2. Message format from iframe → parent

Inside the iframe, `app.emit(type, data)` sends messages up:

```js
emit: function (type, data) {
  if (app.isInsideIframe()) {
    window.parent.postMessage({
      type: type,
      data: data,
      _provider: app.provider,          // 'actualize_player'
      unique_id: "<?php echo $r['qs']['unique_id']; ?>"
    }, '*');
  } else {
    console.log('Not in iframe, debug only');
  }
}
```

The Plyr player wires all relevant events into `app.emit`. Notable cases:

* On **timeupdate**:

  * Computes:

    * `data.currentTime`
    * `data.duration`
    * `data.progress` (currentTime / duration * 100)
  * Tracks watched seconds in `storedProgress` per whole-second slot.
  * Computes:

    * `data.watchedSeconds`
    * `data.watchedPercent` (capped at 100%)
  * Stores as `app.currentInfo`.

* On **loadedmetadata**:

  * Sets `data.duration = app.player.duration`.

In the parent (`video_embed.js`), `handleMessage` checks:

```js
if (!e.data._provider || e.data._provider != 'actualize_player') return;
if (e.data.unique_id != self.unique_id) return; // message for another iframe
```

Then switches on `data.type`. Currently it:

* Calls `options?.events?.ready()` on `type: 'ready'`.
* Calls `options?.events?.loadedmetadata(data.data)` on `type: 'loadedmetadata'`.
* Calls `options?.events?.timeupdate(data.data)` on `type: 'timeupdate'`.
* Logs unsupported events with a warning.

**Note:** The iframe is already emitting a broader set of Plyr events; you can easily forward more of them by extending `handleMessage`.

***

## 6. Analytics & watch-time tracking

Inside `player.php`’s JS, `app.sendIncrement(reason)` collects and sends watch progress:

```js
sendIncrement: function (reason) {
  var payload = {
    current: {
      id:  app.vid + '_' + app.uid,
      vid: app.vid,
      uid: app.uid,
      max_progress: (app.currentInfo && app.currentInfo.watchedPercent)
                      ? app.currentInfo.watchedPercent : 0,
      last_time: app.player.currentTime
    },
    token: app.token
  };

  if (payload.current.max_progress > 100) payload.current.max_progress = 100;

  // Prefer sendBeacon on unload:
  if (navigator.sendBeacon && (reason === 'ended' || reason === 'pagehide')) {
    navigator.sendBeacon(app.apiurl + '/v1/schema/video_view', JSON.stringify(payload));
  } else {
    fetch(app.apiurl + '/v1/schema/video_view', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      keepalive: true,
      body: JSON.stringify(payload)
    });
  }
}
```

**When it sends increments:**

* On `pause`: `app.player.on('pause', () => app.sendIncrement('pause'));`
* On `ended`: `app.player.on('ended', () => app.sendIncrement('ended'));`
* On tab hide / page unload: `app.sendIncrement('pagehide');`
* Heartbeat every 30 seconds _while playing_:

  ```js
  app.int = setInterval(() => {
    if (app.player.playing) {
      app.sendIncrement('heartbeat');
    }
  }, 30000);
  ```

**Payload fields:**

* `current.id` – composite `<vid>_<uid>`.
* `current.vid` – video id.
* `current.uid` – user id.
* `current.max_progress` – max watched percentage (0–100).
* `current.last_time` – last known playback time in seconds.
* `token` – same token as used for auth.

Back-end endpoint: `POST /v1/schema/video_view`.

### Data Example

```json
{
    "_id": "691c0cbb73f9df40d70b7b92",
    "id": "1111301216_UIAMPLAYER1",
    "uid": "UIAMPLAYER1",
    "type": "vimeo",
    "vid": "1111301216",
    "views": 20,
    "last_view": 1763466114,
    "last_time": 0,
    "max_progress": 100
}

```

***

## 7. Usage examples

### 7.1. Minimal embed with default controls

```js
const container = document.getElementById('video-container');

const player = new modules.video_embed({
  renderTo: container,
  type: 'youtube',
  id: 'pK_axPtr23Y',      // YouTube ID
  controls: true
});
```

### 7.2. Embed with custom events and no controls

```js
const player = new modules.video_embed({
  renderTo: document.getElementById('video'),
  type: 'video',
  id: 'my-internal-video-id',
  controls: false,  // adds ?nocontrols=1
  events: {
    ready() {
      console.log('Player ready');
    },
    loadedmetadata(meta) {
      console.log('Duration:', meta.duration);
    },
    timeupdate(info) {
      console.log(
        'Current time:', info.currentTime,
        'Progress:', info.progress,
        'Watched%:', info.watchedPercent
      );
    }
  }
});

// Control via JS:
player.play();
setTimeout(() => player.seek(60), 5000); // jump to 1:00
```

***

## 8. Things to be aware of / future considerations

* **Message security:** You currently use `postMessage` with `"*"` as target origin. For stricter security, consider constraining to a specific origin (using the `$origin` inferred in PHP).
* **Extending events:** The player already emits a wide range of Plyr events; the parent only handles a subset. You can expand `handleMessage` to plumb through more event types without changing `player.php`.
