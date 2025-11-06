---
title: About Phi Framework
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
# What `phi.js` is (at a glance)

`phi.js` is the front-end framework layer that:

- Registers and instantiates **views/components**, manages a **view stack** (pages & overlays), and runs each view’s lifecycle (start/stop/hide/destroy).  
- **Loads dependencies** on demand before a view shows, with retry/fail UX.  
- **Renders templates** with helpers, binds DOM events via attribute syntax (`[action]`, `[link]`), and wires modules via `bind`.   
- Provides a tiny **event bus** (`emit`) and registry helpers (`loadView`, `loadRoute`, `loadDependencies`).  
- Bridges **navigation/external links** including in-app Safari View Controller/Chrome Custom Tabs and keyboard/status-bar hooks.  
- Supports **hot-reload** of components/views. (Views are restarted/refreshed when sources change.) 

# Core concepts

- **Apps / Views / Routes**

  - Register a view class into `window.views[app][component]` with `loadView`, and (optionally) a `route` handler with `loadRoute`. 
  - Instantiate and show a view via `registerView(…options…)` → `startView` (ensures deps, then `component.show(reload)`).  
- **View lifecycle** (defaults on every view)  
  `start()` → `stop()` → `hide()` / `reshow()` → `destroy()` → `deregister()`; hooks trigger and listeners are cleaned up.   
- **Dependency gating**  
  `ensureDependencies([...])` checks/loads code (URL or module/dna), tracks a dependency map, and faults with a user-visible fail flow.  
- **Rendering & data-binding**  
  `render(domLike, { template, data, bindings, … })` finds a template, renders it, auto-mounts nested `[render]` nodes, then binds `[action]`/`[link]` handlers and optional `binding(container)` hook. Helpers: `getOptions`, `hydrateOptions`, `formatOptions`.    

# Public API cheat-sheet (most used)

**Registration & boot**

- `phi.loadView(app, componentId, ctor)` – registers a view class. 
- `phi.loadDependencies(app, componentId, deps[])` – declares per-view deps. 
- `phi.loadRoute(app, componentId, RouteClass)` – registers a route and calls `route.register()`. 
- `phi.registerView(componentId, options[, store, reload, onReady])` – creates instance, applies defaults, attaches IDs/paths/qs, then `startView`. Returns the registry entry.  

**Lifecycle helpers (per-view defaults)**

- `start()` / `stop()` – mark active, fire hooks/events. 
- `hide()` / `reshow()` – show/hide DOM and resume timers; may update route. 
- `refresh(force)` / `reload(force)` – re-render if active (or forced). 
- `destroy(reload)` → `deregister()` – remove DOM, listeners, and view IDs (and ensure the “current view” is valid).  

**Dependency loading**

- `ensureDependencies(deps, ok, fail)` – ensures deps; logs each missing dep; calls `ok()` or `fail()`. 
- `loadDependencyCode(deps, cb)` – URL scripts or offline DNA modules; async parallel; reports processed totals.  
- `isDependencyAvailable(v)` / `dependencyList` – registry for what’s loaded. 

**Render & DOM binding**

- `render(dom, opts)` – template render + mount, run `[render]` children, wire `[action]`, `[link]`. Returns container.   
- `bind(context, ele, "module:opts[,:alias]")` – instantiate modules from attributes. 
- `getNodes(res, css, [no_parent])` – helper for querying parent + descendants. 
- Event plumbing: `actions` map and `handleEvent(e, options, container)` to route to context methods.  

**Navigation / external links**

- Auto-handles `[link]` attributes; on mobile it prefers in-app browser (SVC/Custom Tabs) with event hooks for open/loaded/closed and status bar tinting.  

**Keyboard & UI chrome**

- `onKeyboardWillShow/Hide` control the bottom nav visibility and let current view reset layout. 

**Bus & utilities**

- `emit(channel, data)` broadcasts to registered listeners per-component. 
- `refreshHome()` triggers a hot-reload of the home view. 
- `restart(restart, go_to)` performs a full reload (app vs. site) and can persist a restart path. 

# How a view renders (end-to-end)

1. `registerView()` creates the instance, applies defaults, stores IDs/paths/qs. 
2. `startView()` -> `ensureDependencies()`; if ok, calls your `component.show(reload)` and clears “loading” spinners; else shows a fail alert with retry. 
3. Inside your view’s `render()`, `phi.render()` injects HTML, scans for `[render]` (child views), `[action]`/`[link]` (binding/events), then runs the optional `binding` hook.   

# Hot-reloading

When the dev channel signals a file update, affected views are refreshed through the view delegate; modules hot-reload via `phi.onHotReload`. (You’ll see it used by user logic that calls `modules.viewdelegate.onHotReload(id)` and then `phi.onHotReload(component)`.) 

# Things to watch out for (design notes)

- **Dependency naming & environment flags**: Dependencies prefixed with `[web]` are treated specially; ensure you prefix consistently for mobile/web parity. 
- **Timeouts & retries**: External `<script>` loads use a 6s timeout—if you have slow networks, consider staggered deps or a CDN. 
- **View destroy vs. reload**: On reloads, instances keep the same UUID; on full destroy you must be sure you’ve cleaned up timers/sockets—`destroy()` triggers `deregister()` later to keep the stack sane.  
- **Event binding density**: Attribute-driven `[action]` binding is powerful but can become opaque; keep action names consistent with view methods and prefer `binding(container)` for complex wiring.  
- **External links UX**: In-app browsers set different toolbars/tints on iOS vs Android—verify dark/light status bar pairings per screen.