---
layout: documentation
id: documentation
title: Upgrading from previous versions of Sparkle
---

We strongly recommend upgrading Sparkle to the [latest production release](//github.com/{{ site.github_username }}/Sparkle/releases) because there have been [important security and reliability improvements](/documentation/security-and-reliability). Very old versions of Sparkle also suffer some incompatibilities with the latest macOS versions.

## Upgrading to Sparkle 2.7 (beta)

Sparkle 2.7 (beta) introduces a new format for delta updates, which preserves the creation date of the app bundle and creates slightly more efficient patches. If you don't use `generate_appcast`, please check the [compatibility notes for creating delta updates](/documentation/delta-updates/#compatibility).

[Custom version comparators](/documentation/api-reference/Protocols/SPUUpdaterDelegate.html#/c:objc(pl)SPUUpdaterDelegate(im)versionComparatorForUpdater:) have been deprecated. Please use an increasing (numerical `x`, `x.y`, or `x.y.z`) `CFBundleVersion` / `sparkle:version` instead and disjoint them from more human presentable `CFBundleShortVersionString` / `sparkle:shortVersionString` if needed.

## Upgrading to Sparkle 2.6

Sparkle 2.6 no longer sandboxes the Downloader XPC Service by default. If you enabled this XPC Service, please see the updated [sandboxing guide](/documentation/sandboxing) for more information.

## Upgrading to Sparkle 2.4

Sparkle 2.4 strips debug symbols more aggressively now. Please keep a copy of Sparkle’s debug symbols files (.dSYM) around if you build Sparkle from source yourself instead of using a prepackaged version. Otherwise debug symbols can be downloaded from a Sparkle distribution in our [Releases](https://github.com/sparkle-project/Sparkle/releases) page.

The `-setFeedURL:` method on `SPUUpdater` is also now deprecated. Please see [setting the feed programmatically](/documentation/publishing/#setting-the-feed-programmatically) to migrate away from this API.

## Upgrading to Sparkle 2.3

Sparkle 2.3 now requires macOS 10.13 (High Sierra) or later.

`generate_appcast` now only preserves necessary updates and moves old update files into a `old_updates` directory. Please run `generate_appcast --help` for more details.

Serving updates using DSA only without EdDSA is no longer supported.

## Upgrading to Sparkle 2.2

If you're upgrading from Sparkle 2.0 or 2.1 and sandbox your app, Sparkle 2.2 has renamed the XPC Services referenced in the [sandboxed applications](/documentation/sandboxing) guide. If you have scripts that reference these services you will need to update them.

Sparkle 2.2 provides [gentle update reminders](/documentation/gentle-reminders) which background-running (dockless) applications that schedule update checks at the least need to check out.

The `-s` flag to `generate_appcast` and `sign_update` for passing the private EdDSA key as a command line argument is now deprecated. Please run these tools with `-h` for more information if you were using this option.

## Upgrading to Sparkle 2.1

Sparkle 2.1 introduces a new major format for delta updates, which creates more efficient patches and migrates away from deprecated APIs (xar). If you don't use `generate_appcast`, please check the [compatibility notes for creating delta updates](/documentation/delta-updates/#compatibility).

## Upgrading to Sparkle 2.0

Sparkle 2 now requires macOS 10.11 (El Capitan) or later.

Like Sparkle 1, Sparkle 2 supports Swift Package Manager, CocoaPods, and Carthage package managers. Steps for [integrating with Carthage](/documentation) have been updated however.

The `SUUpdater` class has been deprecated and split up in Sparkle 2, but it is still functional for transitional purposes.

Sparkle 2 includes three new classes / protocols:
* **[SPUUpdater](/documentation/api-reference/Classes/SPUUpdater.html)** - The main API in Sparkle for controlling the update mechanism.
* **[SPUUserDriver](/documentation/api-reference/Protocols/SPUUserDriver.html)** - The API in Sparkle for controlling the user interface & interaction (`SPUStandardUserDriver` is the standard one).
* **[SPUStandardUpdaterController](/documentation/api-reference/Classes/SPUStandardUpdaterController.html)** - A convenient controller class that instantiates a `SPUUpdater` targeting the main application bundle and uses Sparkle's standard user interface (`SPUStandardUserDriver`). This class can also be instantiated in a nib and allows [binding UI](/documentation/preferences-ui#sparkle-2) to it.

If you were previously instantiating a `SUUpdater` in a nib, you will want to adopt `SPUStandardUpdaterController` as shown in the [basic setup](/documentation#2-set-up-a-sparkle-updater-object).

If you were previously instantiating a `SUUpdater` in code, please refer to the [programmatic setup guide](/documentation/programmatic-setup).

The deprecated `SUUpdater` in Sparkle 2 is now a stub that uses both a `SPUUpdater` and `SPUStandardUserDriver`.

`SPUStandardUpdaterController` and `SPUUpdater` do not maintain singleton or global instances unlike the older `SUUpdater`. Code that was previously calling `+[SUUpdater sharedUpdater]` in multiple places will need to change to retrieving and storing the standard updater controller or updater reference to a variable.

Plug-ins that share the same process as their host should prefer to use an out-of-process tool such as [sparkle-cli](/documentation/sparkle-cli) instead of sharing or injecting a Sparkle.framework in their host. Refer to how Sparkle supports [updating other bundles](/documentation/bundles#sparkle-2).

`SPUUpdater` when used directly can:
* Update other Sparkle-based bundles
* Use a custom user interface (`SPUUserDriver`)

[sparkle-cli](/documentation/sparkle-cli) makes use of both these features as an example.

`SPUUpdater` and its delegate `SPUUpdaterDelegate` (unlike `SUUpdater`) do not contain any user-interface or AppKit logic. The UI bits were separated into classes implementing `SPUUserDriver` and its delegates. A developer writing their own updater user interface may choose to build Sparkle with `SPARKLE_BUILD_UI_BITS=0` which strips out the standard UI bits that Sparkle provides out of the box.

Downgrade support was poorly supported in Sparkle 1 (via `SPARKLE_AUTOMATED_DOWNGRADES`) and now removed in Sparkle 2.

The behavior for the `-bestValidUpdateInAppcast:forUpdater:` delegate method on `SPUUpdaterDelegate` has changed. Please review its header documentation for more information. In short:
* Delta updates cannot be returned. A top level item must be returned and Sparkle will pick the most appropriate delta item if available.
* Using this method when [channels](/documentation/publishing#channels) or [other features](/documentation/publishing) can be used instead is discouraged.
* An empty update can now be returned (via `SUAppcastItem.emptyAppcastItem`).
* An update whose version is below the current application's version should not be returned if the current application's version is available in the appcast.
* Sparkle filters update items for minimum/maximum OS version requirements before calling this method now.

Here are the new paths to Sparkle's helper tools (note Autoupdate is now a command line tool and the UI bits moved to Updater.app):
```
Sparkle.framework/Autoupdate (symbolic link to Sparkle.framework/Versions/B/Autoupdate)
Sparkle.framework/Updater.app (symbolic link to Sparkle.framework/Versions/B/Updater.app)
```

Please try to avoid using code-signing scripts that reference these tools, the XPC Services, or the framework though. Most of the time this is not needed if you archive and export your application for distribution and notarization. See our [code signing section in our Sandboxing guide](/documentation/sandboxing/#code-signing) and [Distributing your app](/documentation/#4-distributing-your-app) for more details. Note that the Sparkle 2 framework now also uses `Versions/B/` instead of `Versions/A`.

Sparkle 2 supports [sandboxed applications](/documentation/sandboxing) via integration of XPC Services. Note only sandboxed applications require using and bundling the XPC Services.

If you are migrating from earlier beta versions of Sparkle 2, you may find that some of the XPC Services are now optional and re-signing the services may not be necessary. More recent versions of Sparkle 2 now also include the XPC Services inside the framework bundle. Please read the updated [sandboxing guide](/documentation/sandboxing) for more information.

If you use package (pkg) based updates, please see [Package Updates](/documentation/package-updates) for migration notes. In particular, your appcast items may need to include an appropriate installation type to help Sparkle decide if authorization is needed before starting the installer. This is not needed for bare package updates which we recommend adopting.

Sparkle 2 lets users view your application's [full release notes](/documentation/publishing#full-release-notes) when they check for updates and no new updates are available.

Sparkle 2 enhances support for [major upgrades](/documentation/publishing#major-upgrades).

See [Sparkle 2's APIs](/documentation/api-reference) for more information.

## Upgrading to Sparkle 1.26

Support for serving bare, or non-archived, flat packages (`*.pkg` or `*.mpkg`) has been added in Sparkle 1.26. Please see [Package Updates](/documentation/package-updates) for migration details.

## Upgrading to Sparkle 1.21

Support for EdDSA (ed25519) signatures has been added. We recommend migrating to the new keys.

 1. Run `bin/generate_keys`. It will generate a new EdDSA keypair and print a public key.
 2. Add the public key to your app's `Info.plist` as `SUPublicEDKey` property.

If you're using `generate_appcast` tool, that's all you need.

If you were using manual DSA signing with the `sign_update` script, the script has been moved to `bin/old_dsa_scripts`. The new `sign_update` tool is only for EdDSA keys. To transition to new keys, you may need to use both tools.

Please visit [Migrating to EdDSA from DSA](/documentation/eddsa-migration) for more information.

## Upgrading to Sparkle 1.16

For updates that [use the `.pkg` format](https://sparkle-project.org/documentation/package-updates/), Sparkle no longer shows the full Installer GUI by default. Rename your `.pkg` file to `.sparkle_interactive.pkg` to keep the old behavior.

## Upgrading to SDK 10.11 (affects all versions of Sparkle)

Apps compiled with SDK 10.11 are required to use HTTPS only or configure exceptions to allow HTTP access. If you're not using HTTPS yet, please see [App Transport Security](/documentation/app-transport-security/) to ensure that users on macOS 10.11 will be able to get updates.

## Upgrading to Sparkle 1.14

Sparkle doesn't try to "fix" incorrect URLs in the appcast. URLs with any un-encoded non-ASCII characters need to use URL-encoding, e.g. `<enclosure url="https://example.com/Zürich app.zip">` should be changed to `<enclosure url="https://example.com/Z%C3%BCrich%20app.zip">`.

JavaScript is now disabled by default when showing release notes' HTML. It can be re-enabled by setting `SUEnableJavaScript` in the app's `Info.plist`.

## Upgrading to Sparkle 1.13

To work around a bug in Sparkle versions up to 1.10 we've changed bundle ID of the framework (from `org.andymatuschak.Sparkle` to `org.sparkle-project.Sparkle`). It shouldn't affect normal usage of the framework, so you don't need to do anything unless you've built custom tools dependent on the bundle ID.

## Upgrading to Sparkle 1.11

Sparkle now checks whether the updated version will be able to verify future updates as well. This prevents accidentally updating apps to a version that wouldn't be able to update itself any more.

Updates now must either use DSA keys correctly, or not try to use them at all. Same goes for Apple Code Signing. Sparkle will print to the console verbose messages detailing problems found. Try updating and see Console.app if the updates are rejected.

## Upgrading to Sparkle 1.7

Sparkle has kept the same API since the "classic" version 1.5. It should be a drop-in replacement apart from updated system requirements:

### macOS 10.7+ required

To avoid sending incompatible app to users on macOS 10.6 or older, add to your appcast `<item>`:

```xml
<sparkle:minimumSystemVersion>10.7</sparkle:minimumSystemVersion>
```

### 64-bit, ARC or manual memory management

Apple has dropped support for Garbage Collection. Sparkle now uses ARC and is compatible with apps using ARC or manual memory management.

Sparkle uses modern Objective C runtime, so it does not support PowerPC or the previous 32-bit runtime any more. If your application compiles using a recent version of Xcode it'll run fine with the latest Sparkle.

## About old versions of Sparkle

Apps containing Sparkle version 1.5b6 will not pass code signing verification in macOS El Capitan due to broken symlinks included in the framework.

We've reviewed update verification code and made it stricter in the current versions. If you must use old versions of Sparkle, please use it with appcast feed and update downloads over HTTPS only.

If you use Sparkle older than 1.11.1 we recommend disabling automatic updates, as they may corrupt the app bundle if they happen to run during system shutdown.
