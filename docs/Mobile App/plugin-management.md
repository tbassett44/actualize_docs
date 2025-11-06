---
title: Plugin Management
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
# Plugin Management CLI

`dapp` includes a built-in plugin manager to curate a stable, reproducible Cordova plugin set shared across apps/environments. It operates against the **template repo’s** plugin cache and status file.

## Command syntax

```
dapp [app_name] plugin [action] [arg?]
# [app_name] is usually "actualize"
# [action] ∈ { list | add | enable | enableall | disable | load | remove }
```

## What it touches

- **Template plugin cache:** `<templateDir>/plugins/<plugin-id>/`  
  (each contains the plugin’s `plugin.xml`, `package.json`, etc.)
- **Template status file:** `<templateDir>/plugin_status.json`  
  (tracks enabled/disabled state and last update)
- **Project install (transient):** plugin add/remove via Cordova to discover metadata, then copied into the template cache

***

## Actions

### `list`

Lists all known plugins with status and version.

- Reads the template’s `plugins/` and `plugin_status.json`
- Outputs a table: **Status | Version | Plugin ID**
- Status values:

  - **Enabled** — will be enforced by build steps (e.g., `fixplugins`)
  - **Disabled** — present in cache but not active

**Example**

```bash
dapp actualize plugin list
```

***

### `add <spec>`

Adds a plugin to the **template cache** and marks it **enabled**.

Accepted `<spec>` formats:

- **GitHub URL**: `https://github.com/<author>/<repo>`
- **Pinned tag/branch**: `https://github.com/<author>/<repo>/tree/<tagOrBranch>`
- **With commit hash**: same as above, but will install with `#<commit>`
- **Registry name**: `cordova-plugin-camera` (npm/Cordova registry)

What it does (high level):

1. If GitHub URL:

   - Validates `plugin.xml` exists at the specified tag/branch.
   - Installs with `cordova plugin add <repo>[#commit]` (verbose).
2. If registry name:

   - Removes any prior install and runs `cordova plugin add <name>`.
3. Detects which new folder appeared (`plugins/` or `node_modules/`), resolves **plugin id** from `plugin.xml`.
4. Copies the installed plugin into **`<templateDir>/plugins/<plugin-id>`**.
5. Cleans transient keys in `package.json` (e.g., keys starting with `_`).
6. Updates **`plugin_status.json`** to set the plugin **enabled** and bump `lastUpdate`.

**Examples**

```bash
# Add from registry
dapp actualize plugin add cordova-plugin-camera

# Add from GitHub, pinned to tag
dapp actualize plugin add https://github.com/apache/cordova-plugin-splashscreen/tree/6.0.2

# Add from GitHub repo root (defaults to main), with a specific commit hash
dapp actualize plugin add https://github.com/apache/cordova-plugin-whitelist#<commit>
```

***

### `enable <plugin-id>`

Marks a cached plugin as **enabled** in `plugin_status.json`.

- Does **not** rebuild immediately; takes effect on next `build*` (via `fixplugins`).

**Example**

```bash
dapp actualize plugin enable cordova-plugin-camera
```

***

### `disable <plugin-id>`

Marks a cached plugin as **disabled** in `plugin_status.json`.

- Leaves the plugin in cache but it will be **removed/not installed** during the next build’s plugin sync.

**Example**

```bash
dapp actualize plugin disable cordova-plugin-camera
```

***

### `enableall`

Sets **all** cached plugins to **enabled** in `plugin_status.json`.

**Example**

```bash
dapp actualize plugin enableall
```

***

### `remove`

Currently **not supported** by design. The CLI prints:

> “There is no Remove Function! Use ‘disable’ instead”

Rationale: Retaining plugins in the template cache (but **disabled**) preserves historical pins and speeds up re-enabling.

***

## How this integrates with builds

- During `buildit` / `buildios` / `buildandroid`, the step **`fixplugins`** aligns the working project with the template cache and **enabled** set:

  - installs/pins required plugins,
  - removes extra/stale ones,
  - repairs `package.json` when needed.

This means the **plugin manager** is your single source of truth for plugin composition; the **build** enforces it.

***

## “Holes to poke” & fixes

1. **Missing `loadPlugins()` implementation**

   - The CLI references it, but no handler exists. Without this, `plugin load` breaks.
   - Suggested minimal implementation (below) to:

     - ensure `plugin_status.json` exists,
     - iterate `<templateDir>/plugins/*`,
     - (re)install each plugin into the current project,
     - mark status to **enabled** unless explicitly disabled.

2. **No explicit `remove`**

   - By policy, use `disable` instead. That’s fine, but consider adding `purge <plugin-id>` to delete a plugin from cache **only** if you want a way to clean dead code (guard it behind `--force`).

3. **Version provenance**

   - `add` copies the installed plugin into cache. If the source was a GitHub URL, also persist the **source + tag/commit** alongside (e.g., in `plugin_status.json` or a `sources.json`) so you can reconstruct/verify provenance later.

4. **Atomicity & errors**

   - If `cordova plugin add` succeeds but copying to cache fails, you can get drift. Consider staging to a temp dir and moving atomically.

***

## Patch: add a minimal `loadPlugins()` handler

Here’s a safe minimal version you can drop into `dapp.php` (near your other plugin helpers). It:

- Ensures `plugin_status.json` exists,
- Installs **enabled** plugins from the template cache,
- Optionally supports `--all` to install everything regardless of status.

```php
public static function loadPlugins($installAll = false){
    chdir(self::$templatedir);
    $statusPath = self::$templatedir . '/plugin_status.json';
    if (!is_file($statusPath)) {
        $init = ['created' => time(), 'plugin' => [], 'lastUpdate' => time()];
        file_put_contents($statusPath, json_encode($init, JSON_PRETTY_PRINT));
        self::cl('Initialized plugin_status.json');
    }
    $status = json_decode(file_get_contents($statusPath), true);
    if (!isset($status['plugin'])) $status['plugin'] = [];

    $pluginsDir = self::$templatedir . '/plugins';
    if (!is_dir($pluginsDir)) self::cl('Template plugins directory not found: '.$pluginsDir, 1);

    $dirs = array_filter(scandir($pluginsDir), function($d){
        return $d !== '.' && $d !== '..';
    });

    // Ensure we are in the project home (where cordova commands run)
    chdir(self::$home);

    foreach ($dirs as $pluginId) {
        $pluginPath = $pluginsDir . '/' . $pluginId;
        if (!is_dir($pluginPath) || !is_file($pluginPath.'/plugin.xml')) {
            self::cl('Skipping invalid plugin folder: '.$pluginId);
            continue;
        }
        $isEnabled = isset($status['plugin'][$pluginId]) ? ($status['plugin'][$pluginId] === 'enabled') : false;
        if (!$installAll && !$isEnabled) {
            self::cl('Skipping disabled plugin: '.$pluginId);
            continue;
        }

        // Remove any previous install to avoid duplicates
        passthru(self::getCliType().' plugin rm '.$pluginId.' 2>/dev/null');

        // Add from local path (stable, offline-friendly)
        $cmd = self::getCliType().' plugin add "'.$pluginPath.'" --save';
        self::cl('Installing plugin: '.$pluginId);
        $ret = 0;
        system($cmd, $ret);
        if ($ret !== 0) {
            self::cl('Failed to install plugin: '.$pluginId);
        } else {
            // Mark enabled unless it was explicitly disabled
            if (!isset($status['plugin'][$pluginId]) || $status['plugin'][$pluginId] !== 'disabled') {
                $status['plugin'][$pluginId] = 'enabled';
            }
        }
    }

    $status['lastUpdate'] = time();
    file_put_contents($statusPath, json_encode($status, JSON_PRETTY_PRINT));
    self::cl('Plugin load complete.');
}
```

**Wire it to the CLI:**

- Keep your existing `case 'load': self::loadPlugins(); break;`
- Add support for `--all` by parsing flags in `init()` and passing `true` into `loadPlugins(true)` if present.

**Usage**

```bash
# Load all enabled plugins into the current project
php dapp.php actualize plugin load

# Load ALL plugins from cache (even disabled)
php dapp.php actualize plugin load --all
```

***

## Quick usage recap

```bash
# Inspect current plugin set
dapp actualize plugin list

# Add & enable a plugin (registry)
dapp actualize plugin add cordova-plugin-geolocation

# Add a pinned GitHub plugin
dapp actualize plugin add https://github.com/apache/cordova-plugin-inappbrowser/tree/6.0.0

# Enable/Disable a cached plugin
dapp actualize plugin enable cordova-plugin-camera
dapp actualize plugin disable cordova-plugin-camera

# Bootstrap project from template cache (enabled only)
dapp actualize plugin load
```