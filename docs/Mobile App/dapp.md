---
title: CLI App Builder
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
# dapp.php — Cordova Build Wrapper (CLI)

## What this is

`dapp` is a PHP CLI wrapper around Cordova that standardizes multi-app, multi-environment mobile builds, plugin management, signing, store metadata, and push infrastructure (APNs/SNS/FCM). It bridges **native** capabilities and **JavaScript** via Cordova while enforcing a **single UI codebase** across iOS and Android.

<br />

## Getting Started

Download the dapp repo (ask [juicy@actualize.earth](mailto:juicy@actualize.earth) for access if needed). 

Create a shell shortcut on your system with this logic.  Can be an alias in you .bash_profile or .zshrc or as a file in /usr/local/bin

```
php /Volumes/Projects/dapp/dapp.php $1 $2 $3 $4 $5
```

## CLI usage

```
dapp [app_name] [environment] [action] [forceOrArg?] [flags...]
```

- **app_name**: usually `actualize` (maps to `/config/[app].json`).
- **environment**: e.g. `development`, `production` (used for keystores, backups, conf).
- **action**: see the **Action Flows** below (e.g., `create`, `buildit`, `buildios`, `buildandroid`, `rebuild`, `releaseios`…).
- **forceOrArg?**: optional; several handlers interpret `1`, `2`, or specific strings (e.g., `debug`) to alter behavior.
- **flags**: any `-` or `--` flags are captured (e.g., `--debug`) and used by some steps.

> Internally: `init()` parses args, merges any `env_settings` from `/config/[app].json`, sets paths, then calls `run($action)`.

***

## Action flows (from `run()` → `$commands`)

Below, each action lists the **step keys** in order and what each does (traced into `runCommands()` and helpers).

### 1. `create`

Steps:

- `createstart` → If `force` is truthy, removes existing app home dir and clears iOS/Android certs/keystores; `force==2` wipes **all** keystores for this app/env and commits that change.
- `getconf=check` → Calls `getConfUrl()` and validates the remote app config exists.
- `exec`/`chdir`/`exec` → Creates the env folder and runs `cordova create [abbr]`.
- `getconf=save` → Writes remote app config to `settings.json`, preserving `buildversion` if present.
- `chdir` / `exec` x2 → Creates `res` and `builds`.
- `ensureiosapp` → Uses Fastlane to create the Apple App ID, enable push, and (re)hydrate provisioning/certs cache if available.
- `createsns` → Creates AWS SNS platforms for APNS/APNS_SANDBOX (and later Android).
- `chdir` / cleanup default Cordova `www/*` artifacts.
- `buildit` → Full iOS+Android build pipeline (see below).

### 2. `buildit`  (primary build of both platforms)

Steps:

- `getconf=save` → Refresh `settings.json` from API.
- `incbuild` → Bumps `buildversion` and persists to `settings.json`.
- `chdir` to app home.
- `copyresources` → Creates `res/*` structure and downloads/cooks splash/icons/etc from API; writes `res/version.json` to cache versions.
- `ensurewww` → Populates `www/dist` from template; fetches `boot.js`, optional `library/core` bundles specified by `settings`, default splash, etc.
- `createconfig` → Writes `www/conf.js` from `settings.json` (ids, scheme, flags, API, etc).
- `fixxml type=ios` → Renders `config.xml` from template placeholders for iOS (bundle id, name, versions, FB keys, etc).
- `fixplugins` → Pins/installs/removes Cordova plugins to match the template repo and `android_pins.json`; fixes `package.json`/lock when needed.
- `ensurehooks` → Copies hook scripts from template if version changed; handles iosrtc Swift hooks nuance.
- `buildios` → Installs pods, tweaks iOS build scripts, sets entitlements/schemes, prepares and compiles iOS.
- `fixxml type=android` → Renders Android `config.xml` (different ids/names/versionCode).
- `buildandroid` → Ensures android platform, fixes manifest/build extras, sets keystore & signing, adjusts debug/release mode, compiles Android; fixes android scheme intent and resources.

### 3. `buildios`

Steps:

- `getconf=save` → Refresh config.
- `incbuild`
- `ensurenpm` → Installs Node deps needed for the pipeline (e.g., `xml-entities`, `html-entities`).
- `fixplugins`
- `copyresources`
- `ensurewww`
- `createconfig`
- `fixxml type=ios`
- `ensurehooks`
- `buildios`

### 4. `buildandroid`

Steps:

- `getconf=save`
- `incbuild`
- `ensurewww`
- `ensurenpm` (e.g., `recursive-readdir`)
- `fixplugins`
- `createconfig`
- `fixxml type=android`
- `buildandroid`

### 5. `rebuild` (both platforms from scratch-ish)

Steps:

- `rebuild` → Removes iOS/Android platforms and their folders to force a clean add/build.
- `buildit` → Full pipeline as above.

### 6. `rebuildios`

Steps:

- `rebuildios` → Removes the iOS platform and runs `npm install xcode` (fresh).
- `getconf=save`
- **`updatetemplate`** _(listed but no handler—see “Holes to poke” below)_
- `incbuild`
- `chdir`
- `copyresources`
- `ensurenpm`
- `fixplugins`
- `ensurewww`
- `createconfig`
- `fixxml type=ios`
- **`devcert`** _(listed but no handler—see “Holes to poke” below)_
- `ensurehooks`
- `buildios`

### 7. `rebuildandroid`

Steps:

- `rebuildandroid` → Removes Android platform and `platforms/android`.
- `getconf=save`
- **`updatetemplate`** _(listed but no handler—see “Holes to poke”)_
- `incbuild`
- `chdir`
- `copyresources`
- `ensurenpm`
- `fixplugins`
- `ensurewww`
- `createconfig`
- `fixxml type=android`
- `buildandroid`

### 8. `runios` / `runandroid`

- `runios` → `cordova run ios --device`
- `runandroid` → `cordova run android` (or `--target` emulator if present; see `emulate`)

### 9. `debugios` / `debugandroid`

- `debugios` → Prints `idevicesyslog | grep [bundleId]` to tail logs for your app.
- `debugandroid` → Installs debug APK and prints the `adb logcat` command filtered to your PID.

### 10. `listemulators` / `emulate`

- `listemulators` → Runs Cordova’s Android emulator lister.
- `emulate` → Picks a default emulator (configurable), runs `cordova run android --target=...` (or plain `run android` if `force` set).

### 11. `releaseios` / `releaseandroid` (store publish)

- `releaseios` → If `builds/ios-release.ipa` exists, sets iOS metadata files and runs `fastlane deliver` to upload. Growl notification on completion.
- `releaseandroid` → If Play metadata exists, runs `fastlane supply` to upload AAB/APK. Otherwise suggests running `androidmetadata`.

### 12. Metadata & screenshots

- `updatemetadata` → Writes iOS metadata files (description/keywords/support/copyright/privacy/etc) then `fastlane deliver run`.
- `androidmetadata` → Initializes/updates Play metadata tree under `builds/android_settings/metadata` and writes values from `settings.json`.
- `loadscreenshots` → Fetches iOS and Android screenshots from your API, stores at expected paths for App Store/Play Console.
- `uploadscreenshots` → (commented out, left as scaffold)

### 13. Push & certificates

- `ensureiosapp` → Fastlane `produce` to create App ID (optionally sets company), enables push, restores cached certs where possible, and ensures `.p12/.pem/.pkey` artifacts exist.
- `ensureProvisionProfile` → Fastlane `sigh` to pull provisioning profile; caches it, and marks for git commit.
- `updatepushcert` → Fastlane `pem` (production APNs cert), converts to PKCS8 key/cert via OpenSSL, copies into keystore cache.
- `ensuresandboxcreds` → Fastlane `pem --development` (sandbox APNs), converts formats, copies to keystore cache.
- `createsns` → AWS SDK: creates APNS / APNS_SANDBOX platform apps (and later with `androidsns` for GCM/FCM) using the certs/keys you just created.
- `updatesns` → Finds existing SNS platform app ARNs and updates their credentials with the newly generated key/cert.
- `androidsns` → Creates the Android GCM/FCM platform app in SNS using your server key.
- `testpush` → Opens a TLS session to the APNs gateway with your cert to validate it.
- `pushdate` → Prints the expiration of the APNs cert via `openssl x509 -enddate`.

### 14. Utilities

- `copyresources` → Builds out `res/*`, calls your image endpoints, generates app icons/splash, copies defaults if missing, and version-caches assets.
- `ensurewww` → Populates `www/dist/*` from template; fetches `boot.js`, optional library/core bundles (JS/CSS/templates/fonts).
- `createconfig` → Produces `www/conf.js` from `settings.json` (ids, API, scheme, toggles).
- `fixxml type=ios|android` → Renders `config.xml` placeholders from template and `settings.json` (ids, names, versionCode, FB keys, YouTube key, etc).
- `fixplist` → Renders iOS Info.plist from template with bundle id, app version, FB keys, etc.
- `fixplugins` → Ensures plugins match the template + pinned versions; removes old/extra plugins; repairs `package*.json` if needed.
- `ensurehooks` → Copies hook scripts from template when version differs; accounts for iosrtc Swift hook duplication.
- `buildios` → Ensures iOS platform and pods, fixes target scripts, sets entitlements/schemes, prepares and builds.
- `buildandroid` → Ensures Android platform, fixes Gradle/manifest, installs signing (`build-extras.gradle`), ensures URL scheme intent filter, compiles; then post-build fixes (icons, sounds, etc).
- `setkeystore` → Generates Android keystore & `release-signing.properties` (using config-derived subject & passwords), caches keystore in template repo’s `/keystores`.
- `ensureAndroidPins` → Removes any installed plugin whose pinned version changed, then persists `android_pins.json` locally.
- `ensureAndroidScheme` → Warns if your custom URL scheme intent filter is missing from `AndroidManifest.xml`.
- `setprojectid` → Calls `fastlane produce` & `produce --sku` to get/store the App Store “project id” (sets `ios_store_id`).
- `match` → Fastlane match appstore sync for iOS signing.
- `backup` → Copies `ios-release.ipa` and Android artifacts into a backup directory; if `force==='debug'` or `--debug` is set, it will also S3-upload the debug APK and print a QR to install.
- `hash` → Prints base64 SHA-1 hash of Android keystore (useful for Google OAuth console).
- `listemulators` / `emulate` → Emulator helpers.
- `runios` / `runandroid` / `debug*` → Run and log helpers.
- `commit` / `finish` → Stages and pushes any keystore/provisioning changes to the template repo (see **Security & Process** notes below).
- `copyprofile` → Restores & installs `ios.mobileprovision`.
- `testaws` → Quick AWS SNS check (wrapper).
- `openxcode` → Opens the `.xcworkspace` directly.

***

## Key Reference (alphabetical, quick definitions)

- **androidmetadata**: Write/update Play metadata files under `builds/android_settings/metadata` from `settings.json`.
- **androidsns**: Create SNS GCM/FCM platform app with server key.
- **backup**: Archive build outputs (and optionally S3-upload debug APK + QR).
- **buildandroid**: Prepare, sign, and build Android; fix manifest/resources; ensure scheme.
- **buildios**: Prepare pods/build settings, entitlements/schemes; build iOS; produce `.ipa`.
- **buildit**: Full iOS+Android pipeline (see Action #2).
- **chdir/exec**: Shell plumbing inside the pipeline.
- **commit/finish**: Git-commit keystore/provisioning updates and push.
- **copyprofile**: Restore/install provisioning profile from cache.
- **copyresources**: Fetch/cook icons, splash, and other assets into `res/*`.
- **createconfig**: Generate `www/conf.js` from `settings.json`.
- **createsns**: Create APNS/APNS_SANDBOX SNS platform apps with current certs/keys.
- **createstart**: Clear existing app home and keystores based on `force`.
- **debugandroid/debugios**: Show how to tail logs (and install debug APK on Android).
- **emulate/listemulators**: Run emulator and list available images.
- **ensurehooks**: Copy/update Cordova hooks from template.
- **ensureProvisionProfile**: Fetch provisioning, cache and mark for commit.
- **ensurewww**: Populate `www/dist` and fetch `boot.js`, library/core bundles.
- **ensurenpm**: Install Node deps required by the pipeline.
- **ensuresandboxcreds**: Generate dev (sandbox) APNs certs/keys via Fastlane + OpenSSL.
- **fixplist**: Render iOS Info.plist from template.
- **fixplugins**: Sync plugin set & versions; clean stale entries; handle edge cases.
- **fixsns**: If `force` is set, delete old APNs artifacts so a clean set can be created.
- **fixxml (type=ios|android)**: Render `config.xml` placeholders.
- **getconf=(check|save)**: Validate and/or save `settings.json` from your conf API.
- **hash**: Compute base64 SHA-1 of Android keystore.
- **incbuild**: Increment `buildversion` in `settings.json`.
- **match**: Fastlane `match appstore` for iOS cert sync.
- **openxcode**: Open the workspace in Xcode.
- **publishandroid/releaseandroid**: Upload Android to Play (via `supply`).
- **releaseios**: Upload iOS to App Store (via `deliver`).
- **rebuild / rebuildios / rebuildandroid**: Remove platforms then rebuild.
- **runandroid/runios**: Cordova run on device/emulator.
- **setprojectid**: Create/set App Store project id (SKU + display name).
- **setvoip**: (VoIP push) Regenerate VOIP PKCS8 key and update SNS voip platform (requires `force` to clean old files first).
- **shouldBuild**: Compare local vs remote conf, trigger build if changed.
- **testaws**: Connectivity/sanity for AWS bits.
- **testpush**: APNs connectivity test using your cert/key.
- **updateios**: Remove/add iOS platform (refresh).
- **updatemetadata**: Write iOS metadata files and run `deliver run`.
- **updatepushcert**: Generate prod APNs cert/key and copy to keystore cache.
- **updatesns**: Update SNS platform credentials with new cert/key.
- **uploadscreenshots**: (Scaffold) Would upload iOS screenshots.

***

## Multiple apps & environments

- `dapp` keeps independent **per-app** configs (`/config/[app].json`) and keystores under `template/keystores/[env]/[app]`.
- You can build **production** and **development** variants by switching the `environment` and the conf served by your API.
- The dispatcher supports **[all]** as `app_name` (in `init()`) to iterate apps for certain actions (build/check).

***

## Security & process considerations (holes to poke)

1. **Two step keys have no handler**

   - `updatetemplate` appears in several action flows but there is **no** `if (isset($v['updatetemplate']))` branch in `runCommands()`. It’s a no-op today. If you intended to refresh the app template, add a handler.
   - `devcert` appears in `rebuildios` but there is **no** handler. If you meant “generate development cert” (like `ensuresandboxcreds`), wire it up or replace with the existing step.

2. **`ensureiosapp` conditional**  
   In `runCommands()` it only runs when `isset($v['ensureiosapp']) && self::$force != 2`. If you pass `force==2` to `create`, the earlier `createstart` nukes keystores—but `ensureiosapp` then _won’t_ rebuild certs. If that’s intentional, great; otherwise you’ll strand a fresh app without certs.

3. **Secrets committed to git**  
   `commit()`/`finish()` run `git add . && git commit && git push` inside the **template** repo, and you cache keystores/profiles there. Make sure that repo is private, access-controlled, and never mirrored publicly. Consider splitting secrets to a dedicated secrets store or locked repo.

4. **`self::$confurl` provenance**  
   `getConfUrl()` concatenates `self::$confurl.$id.'/conf?...'`, but `self::$confurl` isn’t obviously set in this file. Ensure your `/config/[app].json` or bootstrap sets it before the first `getconf` call, or the initial `create` will fail.

5. **Toolchain versions drift**  
   iOS: Cordova iOS pinned at `ios@7.0.1`, custom Podfile patches, custom `fixios_target.js`. Android: Gradle/AGP changes can break builds. Consider centralizing version pins in your template config and surfacing a `dapp doctor` check.

6. **`rm -rf` safety**  
   Multiple destructive deletes (platform folders, keystores, images). You already scope paths, which is good. Consider a `--dry-run` and/or an interactive confirm when `force==2`.

7. **Network dependencies**  
   Steps like `ensurewww`, `copyresources`, `loadscreenshots`, and Fastlane/SNS calls fail hard without network. Consider retries + clearer error messages.

8. **Pinning & plugin health**  
   `fixplugins` is doing a lot (reading template `plugins/*/plugin.xml`, comparing pins, editing `package.json`). Add a summary report (plugins added/removed/updated) and a “lock” snapshot to aid reproducibility.

9. **APK vs AAB**  
   You build AAB then make an APK with bundletool in backup. Ensure the Play upload path is AAB-first and that APK creation is only for sideload/debug.

10. **Schema/URL-scheme checks**  
    `ensureAndroidScheme()` only warns; consider auto-inserting the intent filter when missing.

***

## Example quick usage

```bash
# Fresh app bootstrap & both-platform build (development)
dapp actualize development create

# Rebuild iOS cleanly and compile
dapp actualize development rebuildios

# Android only, bump build and compile (production)
dapp actualize production buildandroid

# Publish to App Store / Play (after successful builds)
dapp actualize production releaseios
dapp actualize production releaseandroid

# Test APNs certificate validity
dapp actualize production testpush

# Get the configured conf API URL for this app
dapp actualize development getapi
```