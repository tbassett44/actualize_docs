---
title: Introduction
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
## Introduction to the Mobile App CLI Builder

Our mobile app CLI builder is a wrapper around the Cordova build tools, designed to streamline development while preserving flexibility. At its core, **Cordova provides the essential bridge between native device functionality and JavaScript**, allowing developers to write application logic in one language and still access native APIs (camera, contacts, notifications, etc.).

### Why This Matters

By leveraging Cordova, we gain the ability to:

* **Write once, run everywhere**: A single UI codebase that adapts across iOS and Android devices, dramatically reducing development and maintenance overhead.
* **Access device features through plugins**: Cordova plugins expose native functionality in a consistent JavaScript API, removing the need to write custom Objective-C, Swift, Java, or Kotlin code for each platform.
* **Maintain a unified workflow**: Teams can build and iterate quickly without managing multiple native projects.

### The Role of `dapp`

To simplify working with Cordova directly, we use our wrapper tool called **`dapp`**. This command-line tool standardizes builds, abstracts away common pitfalls, and integrates our development practices into a cohesive workflow. With `dapp`, you get a consistent interface to:

* Configure and run builds
* Manage plugins
* Handle platform-specific quirks
* Ensure reproducible builds across environments

### Key Benefits of `dapp`

* **Production vs. Development Builds**: With `dapp`, you can generate both production-ready apps (optimized, signed, and store-ready) and lightweight development builds for rapid testing. This dual-mode capability saves significant time and reduces errors when switching between contexts.
* **Multiple App Management**: `dapp` can manage multiple apps from a single tool, making it easier for teams working across different projects or white-label deployments. Instead of juggling multiple Cordova configurations manually, everything can be orchestrated consistently through `dapp`.

### Things to Be Aware Of

Developing in the Cordova ecosystem comes with a few important considerations:

* **Plugin ecosystem quality varies** — not all plugins are equally maintained, and conflicts between plugins can occur.
* **Platform differences** — while Cordova smooths out a lot of complexity, there are still occasional edge cases where iOS and Android behave differently.
* **Build dependencies** — native build tools (Xcode, Android SDK) must be properly installed and configured. `dapp` helps manage this, but developers still need the underlying toolchains.
