---
layout: documentation
id: documentation
title: Upgrading from previous versions of Sparkle
---

We strongly recommend upgrading Sparkle to the [latest version](//github.com/{{ site.github_username }}/Sparkle/releases), as there have been important fixes in reliability and [security](/documentation/security) of updates. Very old versions of Sparkle also suffer some incompatibilities with the latest macOS versions.

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
