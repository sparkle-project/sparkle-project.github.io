---
layout: documentation
id: documentation
title: Upgrading from previous versions of Sparkle
---

We strongly recommend upgrading Sparkle to the [latest production release](//github.com/{{ site.github_username }}/Sparkle/releases), as there have been important fixes in reliability and [security](/documentation/security) of updates. Very old versions of Sparkle also suffer some incompatibilities with the latest macOS versions.

## Upgrading from Sparkle 1.x to 2.0 (Beta)

**Note**: Sparkle 2.0 is in a pre-release / beta state and not production ready yet.

The `SUUpdater` class has been deprecated and split up in Sparkle 2.x, but it is still functional for transitional purposes.

Sparkle 2.x includes three new classes / protocols:
* **SPUUpdater** - The main API in Sparkle for controlling the update mechanism.
* **SPUUserDriver** - The API in Sparkle for controlling the user interface & interaction (`SPUStandardUserDriver` is the standard one).
* **SPUStandardUpdaterController** - A controller class that instantiates a `SPUUpdater` using `SPUStandardUserDriver` in a nib and allows [binding UI](/documentation/preferences-ui#sparkle-2x-beta) to it.

If you were previously instantiating a `SUUpdater` in a nib, you will want to adopt `SPUStandardUpdaterController` as shown in the [basic setup](/documentation#2-set-up-a-sparkle-updater-object).

If you were previously instantiating a `SUUpdater` in code, you will want to adopt instantiating a `SPUUpdater`.

The deprecated `SUUpdater` in 2.x is now a stub that uses both a `SPUUpdater` and `SPUStandardUserDriver`.

If you create a `SPUUpdater` instance programatically, you can now create an updater that can update other Sparkle-based bundles and/or an updater that can use your own `SPUUserDriver` / user interface. [sparkle-cli](/documentation/sparkle-cli) makes use of both features as an example.

`SPUUpdater` and its delegate `SPUUpdaterDelegate` (unlike `SUUpdater`) do not contain any user-interface or AppKit logic. The UI bits were separated into classes implementing `SPUUserDriver` and its delegates. A developer writing their own updater user interface may choose to use the new `SparkleCore` framework which strips out the UI bits that Sparkle provides out of the box.

`SPUUpdater` does not maintain singleton or global instances (unlike `SUUpdater`). Plug-ins that share the same process as their host should prefer to use an external tool like [sparkle-cli](/documentation/sparkle-cli) instead, rather than sharing or injecting a Sparkle.framework in its host. A bit more details about updating bundles [here](/documentation/bundles#sparkle-2x-beta).

If you have scripts that reference Sparkle.framework's helper tools, here are the new paths (note Autoupdate is now a command line tool and the UI bits moved to Updater.app):
```
Sparkle.framework/Versions/A/Resources/Autoupdate
Sparkle.framework/Versions/A/Resources/Updater.app/
```

Additionally, if you use the `InstallerLauncher` XPC Service, these helpers are in:
```
org.sparkle-project.InstallerLauncher.xpc/Contents/MacOS/Autoupdate
org.sparkle-project.InstallerLauncher.xpc/Contents/MacOS/Updater.app/
```

(In the case of using the `InstallerLauncher` XPC Service, the helpers in Sparkle.framework are unused and can optionally be removed; note in this case you may need to re-sign Sparkle.framework)

Please see the additional setup on using XPC Services and using Sparkle in [sandboxed applications](/documentation/sandboxing). Note using the XPC Services are only required for sandboxed applications, which Sparkle 1.x didn't support.

If you use package (pkg) based updates, please see [Package Updates](/documentation/package-updates) for migration notes. In particular, your appcast items must now include an appropriate installation type now to help Sparkle decide if authorization is needed ahead of time.

See [Sparkle 2.x's APIs](/documentation/customization#sparkle-2x-apis-beta) for more information.

## Upgrading from Sparkle 1.20 and older

Support for EdDSA (ed25519) signatures has been added. We recommend migrating to the new keys.

 1. Run `bin/generate_keys`. It will generate a new EdDSA keypair and print a public key.
 2. Add the public key to your app's `Info.plist` as `SUPublicEDKey` property.

If you're using `generate_appcast` tool, that's all you need.

If you were using manual DSA signing with the `sign_update` script, the script has been moved to `bin/old_dsa_scripts`. The new `sign_update` tool is only for EdDSA keys. To transition to new keys, you will need to use both tools.

## Upgrading from Sparkle 1.15 and older

For updates that [use the `.pkg` format](https://sparkle-project.org/documentation/package-updates/), Sparkle no longer shows the full Installer GUI by default. Rename your `.pkg` file to `.sparkle_interactive.pkg` to keep the old behavior.

## Upgrading to SDK 10.11 (affects all versions of Sparkle)

Apps compiled with SDK 10.11 are required to use HTTPS only or configure exceptions to allow HTTP access. If you're not using HTTPS yet, please see [App Transport Security](/documentation/app-transport-security/) to ensure that users on macOS 10.11 will be able to get updates.

## Upgrading from Sparkle 1.13 and older

Sparkle doesn't try to "fix" incorrect URLs in the appcast. URLs with any un-encoded non-ASCII characters need to use URL-encoding, e.g. `<enclosure url="https://example.com/Zürich app.zip">` should be changed to `<enclosure url="https://example.com/Z%C3%BCrich%20app.zip">`.

JavaScript is now disabled by default when showing release notes' HTML. It can be re-enabled by setting `SUEnableJavaScript` in the app's `Info.plist`.

## Upgrading from Sparkle 1.12 and older

To work around a bug in Sparkle versions up to 1.10 we've changed bundle ID of the framework (from `org.andymatuschak.Sparkle` to `org.sparkle-project.Sparkle`). It shouldn't affect normal usage of the framework, so you don't need to do anything unless you've built custom tools dependent on the bundle ID.

## Upgrading from Sparkle 1.10 and older

Sparkle now checks whether the updated version will be able to verify future updates as well. This prevents accidentally updating apps to a version that wouldn't be able to update itself any more.

Updates now must either use DSA keys correctly, or not try to use them at all. Same goes for Apple Code Signing. Sparkle will print to the console verbose messages detailing problems found. Try updating and see Console.app if the updates are rejected.

## Upgrading from Sparkle 1.6 and older

Sparkle has kept the same API since the "classic" version 1.5. It should be a drop-in replacement apart from updated system requirements:

### macOS 10.7+ required

To avoid sending incompatible app to users on macOS 10.6 or older, add to your appcast `<item>`:

    <sparkle:minimumSystemVersion>10.7</sparkle:minimumSystemVersion>

### 64-bit, ARC or manual memory management

Apple has dropped support for Garbage Collection. Sparkle now uses ARC and is compatible with apps using ARC or manual memory management.

Sparkle uses modern Objective C runtime, so it does not support PowerPC or the previous 32-bit runtime any more. If your application compiles using a recent version of Xcode it'll run fine with the latest Sparkle.

## About old versions of Sparkle

Apps containing Sparkle version 1.5b6 will not pass code signing verification in macOS El Capitan due to broken symlinks included in the framework.

We've reviewed update verification code and made it stricter in the current versions. If you must use old versions of Sparkle, please use it with appcast feed and update downloads over HTTPS only.

If you use Sparkle older than 1.11.1 we recommend disabling automatic updates, as they may corrupt the app bundle if they happen to run during system shutdown.
