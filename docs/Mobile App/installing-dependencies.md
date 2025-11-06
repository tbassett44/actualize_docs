---
title: Installing Dependencies
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
# Dependencies & Installation

`dapp` wraps Cordova and several native toolchains. To build reliably, make sure the following are installed and pinned to sane versions.

## OS & build matrix

* **iOS builds:** *macOS only* (Apple requires Xcode).
* **Android builds:** macOS, Linux, or Windows (WSL works, but native is simpler).

> Recommended: macOS 13+ (Ventura or newer) with Apple Silicon or Intel.

***

## Core dependencies (what & why)

### System & language

* **Node.js (LTS) + npm** – Cordova CLI and many build scripts.
* **Java JDK (17 recommended)** – Android Gradle builds and signing tools.
* **Ruby (system or rbenv) + Bundler** – Fastlane (iOS code signing / App Store) and some utilities.
* **Homebrew (macOS)** – Package manager to install the rest.

### Mobile toolchains

* **Xcode + Command Line Tools (macOS)** – iOS SDK, compilers, simulators.
* **CocoaPods** – iOS dependency manager (Cordova iOS platform uses pods).
* **Android Studio** – SDK Manager, Platform Tools (adb), Build Tools, Android SDK.

### Cordova & project-level

* **Cordova CLI** – Platform & plugin management, build orchestrator.
* **Gradle / Android SDK Build-Tools** – Android build system (installed via Android Studio).
* **Fastlane** – iOS/Android store delivery, provisioning, screenshots.
* **OpenSSL** – APNs cert/key conversions (`updatepushcert`, etc.).
* **AWS CLI** – Optional but useful for SNS validation and S3 uploads.
* **libimobiledevice (optional)** – iOS device logging (`idevicesyslog`).

### npm packages used by the pipeline (install on demand)

* `xml-entities`, `html-entities`, `recursive-readdir` (used by `ensurenpm` in dapp).

### Optional / helpful

* **bundletool** – Generates APKs from AABs for sideloading/debug.
* **jq** – JSON CLI processor (debugging configs).

***

## Installation (macOS)

### 1. Base toolchain

```bash
# Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Node LTS
brew install node

# Java (JDK 17)
brew install openjdk@17
sudo ln -sfn /usr/local/opt/openjdk@17/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk
echo 'export JAVA_HOME=$(/usr/libexec/java_home -v 17)' >> ~/.zshrc
echo 'export PATH="$JAVA_HOME/bin:$PATH"' >> ~/.zshrc

# Ruby helpers (optional but recommended)
brew install rbenv
rbenv init - zsh >> ~/.zshrc
# (restart shell) then optionally: rbenv install 3.2.4 && rbenv global 3.2.4
```

### 2. iOS stack

```bash
# Xcode: install from App Store, then:
xcode-select --install
sudo xcodebuild -license accept

# CocoaPods
sudo gem install cocoapods
pod --version
```

### 3. Android stack

```bash
# Android Studio: install from https://developer.android.com/studio
# Then open Android Studio → SDK Manager and install:
# - Android SDK Platform (target you support)
# - Android SDK Build-Tools (e.g., 34.x)
# - Android SDK Platform-Tools (adb)
# - Android Command-line Tools (latest)

# Environment (add to ~/.zshrc)
echo 'export ANDROID_HOME=$HOME/Library/Android/sdk' >> ~/.zshrc
echo 'export PATH="$ANDROID_HOME/emulator:$ANDROID_HOME/platform-tools:$ANDROID_HOME/tools:$PATH"' >> ~/.zshrc
```

### 4. Cordova, Fastlane, AWS, OpenSSL, helpers

```bash
# Cordova CLI
npm install -g cordova

# Fastlane (via RubyGems)
sudo gem install fastlane

# OpenSSL (new macOS ships one; install if needed)
brew install openssl

# AWS CLI (optional but recommended for SNS/S3 operations)
brew install awscli

# iOS device logging (optional)
brew install libimobiledevice

# Optional helpers
brew install jq
brew install --cask android-platform-tools  # if you want adb via brew
```

### 5. npm modules used by `dapp` (installed on demand)

`dapp`’s `ensurenpm` will install these into your project when needed:

```bash
npm install xml-entities html-entities recursive-readdir --save-dev
```

***

## Installation (Linux)

* Install Node 18/20 LTS + npm via distro or NodeSource.
* Install **JDK 17** (Temurin/OpenJDK) and set `JAVA_HOME`.
* Install **Android Studio** & SDKs; set `ANDROID_HOME`, add `platform-tools` to `PATH`.
* **CocoaPods / Xcode are not available** → Android builds only on Linux.
* Install Cordova and Fastlane:

```bash
sudo npm install -g cordova
sudo gem install fastlane
```

* Install OpenSSL, AWS CLI via your package manager.

***

## Installation (Windows / WSL)

* **Android only** on Windows. Install Node LTS and JDK 17 (Adoptium).
* Install **Android Studio** + SDKs; add `platform-tools` to PATH.
* Install Cordova (`npm i -g cordova`).
* For Fastlane, Windows support is limited; prefer macOS for iOS automation.
* WSL can work for Android but device/USB passthrough can be fiddly—native Windows often simpler for adb.

***

## Version checks & pins

```bash
node -v          # v18.x or v20.x
npm -v
java -version    # openjdk 17.x
cordova -v       # cordova CLI
pod --version    # CocoaPods
fastlane --version
xcodebuild -version
adb version
```

> Tip: Pin **Cordova platforms** in your repo (e.g., `cordova-ios@7.0.1` and a specific `cordova-android`), and keep a “Tooling Versions” table in your README to avoid drift. `dapp` can also log the versions it used on each build for reproducibility.

***

## Apple Developer setup (for iOS builds & TestFlight)

1. **Apple Developer account** with access to Certificates, Identifiers & Profiles.
2. In App Store Connect, set up your app entry.
3. `fastlane` will handle:

   * App ID creation (`produce`)
   * Provisioning profiles (`sigh` / `match`)
   * APNs certs (`pem`)
   * Uploads (`deliver`)

> If you’re using `dapp`’s `ensureiosapp`, `ensureProvisionProfile`, `updatepushcert`, etc., make sure your Apple credentials are available to Fastlane (via `FASTLANE_USER`, `FASTLANE_PASSWORD` or App Store Connect API key JSON, and any 2FA session as required).

***

## Google Play setup (for Android release)

1. **Google Play Console** account with a new app created.
2. Upload path uses **AAB** (preferred). `dapp` can also generate an APK from the AAB with bundletool for sideload/debug.
3. If you use AWS SNS for push on Android, you’ll need your **FCM server key** for `androidsns`.

***

## AWS SNS (push) prerequisites (optional but supported)

* AWS account + IAM user with SNS permissions.
* Configure AWS CLI credentials (`aws configure`).
* `dapp` actions `createsns`, `updatesns`, `androidsns` expect:

  * iOS: APNs cert + key (converted via OpenSSL).
  * Android: FCM server key.

***

## Environment variables (summary)

Add these to your shell profile (e.g., `~/.zshrc`):

```bash
# Java 17
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
export PATH="$JAVA_HOME/bin:$PATH"

# Android SDK (macOS default)
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH="$ANDROID_HOME/emulator:$ANDROID_HOME/platform-tools:$ANDROID_HOME/tools:$PATH"

# Fastlane (if using rbenv)
export GEM_HOME="$HOME/.rbenv/versions/$(rbenv version-name)/lib/ruby/gems/$(ruby -e 'puts RUBY_VERSION')"
export PATH="$HOME/.rbenv/shims:$PATH"
```

Reload your shell after editing:

```bash
source ~/.zshrc
```

***

## Cordova platforms & plugins

Install platforms pinned to your template:

```bash
# From your app home
cordova platform add ios@<pin>
cordova platform add android@<pin>

# (dapp’s `fixplugins` will align plugins to your template/pins)
```

> **Plugin caveat:** Some plugins ship native code that lags behind new Xcode / AGP versions. Keep pins current, and run `dapp fixplugins` (or a full `buildit`) after bumps.

***

## Quick verification & first build with `dapp`

```bash
# Pull app settings and do a clean multi-platform build (development)
php dapp.php actualize development create

# Or: Android-only build (production)
php dapp.php actualize production buildandroid

# Run on device
php dapp.php actualize development runios
php dapp.php actualize development runandroid
```

If anything fails, check:

* `xcodebuild -showsdks` (iOS SDK present)
* `adb devices` (Android device/emulator visible)
* `pod install` inside `platforms/ios` (pods resolve)
* `JAVA_HOME`, `ANDROID_HOME` are set
* Apple account / Fastlane auth (for provisioning/deliver)
* Network access (templates, screenshots, API config)

***

## Troubleshooting & considerations (poking holes)

* **Toolchain drift:** New Xcode/AGP releases can break builds; pin platform versions and keep a “known-good” matrix.
* **Apple Silicon quirks:** Some native gems/pods may need Rosetta or updated binaries. If pods fail, try `sudo gem install ffi -- --enable-libffi-alloc`.
* **JDK mismatch:** AGP 8.x generally prefers JDK 17; using 11 or 21 can cause Gradle toolchain errors.
* **CocoaPods repo slow/locked:** Run `pod repo update` or `pod install --repo-update`.
* **APNs key/cert format:** `dapp` uses OpenSSL to convert; ensure OpenSSL is present and on `PATH`.
* **CI/CD:** For reproducible builds, capture `node`, `cordova`, platform pins, CocoaPods specs repo snapshot, and cache Gradle/pods between runs.

***

If you want, I can tailor this to your exact template pins (e.g., `cordova-ios@7.0.1`, specific AGP/Gradle versions) and add a one-shot “doctor” script that validates the environment before `dapp` runs.
