---
layout: documentation
id: documentation
title: Upgrading from previous versions of Sparkle
---

We strongly recommend using Sparkle 1.11.1 or later, as there have been major improvements to reliability and integrity of updates. Very old versions of Sparkle also suffer some incompatibilities with the latest OS X versions.

If you're upgrading from a very old version of Sparkle we recommend using [Sparkle 1.13 or later](https://github.com/{{ site.github_username }}/Sparkle/releases). The API is backwards-compatible.

## Upgrading to SDK 10.11 (affects all versions of Sparkle)

Apps compiled with SDK 10.11 are required to use HTTPS only or configure exceptions to allow HTTP access. If you're not using HTTPS yet, please see [App Transport Security](/documentation/app-transport-security/) to ensure that users on OS X 10.11 will be able to get updates.

## Upgrading from Sparkle 1.12 and older

To work around a bug in Sparkle versions up to 1.10 we've changed bundle ID of the framework (from `org.andymatuschak.Sparkle` to `org.sparkle-project.Sparkle`).

It shouldn't affect normal usage of the framework, so you don't need to do anything unless you've built custom tools dependent on the bundle ID.

## Upgrading from Sparkle 1.10 and older

Sparkle now checks whether the updated version will be able to verify future updates as well. This prevents accidentally updating apps to a version that wouldn't be able to update itself any more.

Updates now must either use DSA keys correctly, or not try to use them at all. Same goes for Apple Code Signing.

Sparkle will print to the console verbose messages detailing problems found. Try updating and see Console.app if the updates are rejected.

## Upgrading from Sparkle 1.6 and older

Sparkle has kept the same API since the "classic" version 1.5. It should be a drop-in replacement apart from updated system requirements:

### Mac OS X 10.7+ required

To avoid sending incompatible app to users on Mac OS 10.6 or older, add to your appcast `<item>`:

    <sparkle:minimumSystemVersion>10.7</sparkle:minimumSystemVersion>

### 64-bit, ARC or manual memory management

Apple has dropped support for Garbage Collection. Sparkle now uses ARC and is compatible with apps using ARC or manual memory management.

Sparkle uses modern Objective C runtime, so it does not support PowerPC or the previous 32-bit runtime any more. If your application compiles using a recent version of Xcode it'll run fine with the latest Sparkle.

## About old versions of Sparkle

Apps containing Sparkle version 1.5b6 will not pass code signing verification in OS X El Capitan due to broken symlinks included in the framework.

We've reviewed update verification code and made it stricter in the current versions. If you must use old versions of Sparkle, please use it with appcast feed and update downloads over HTTPS only.

If you use Sparkle older than 1.11.1 we recommend disabling automatic updates, as they may corrupt the app bundle if they happen to run during system shutdown.
