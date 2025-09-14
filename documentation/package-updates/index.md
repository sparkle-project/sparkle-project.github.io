---
layout: documentation
id: documentation
title: Package Updates
---
## Package Updates

Package installation allows Sparkle to update your application by downloading and installing a package, `pkg`, or multi-package, `mpkg` without user interaction except for asking for an administrator password.

Note package installation should only be used for apps with very custom installation needs that cannot be satisfied by distributing a regular app bundle. For Sparkle, the downsides of using package updates are:

* Installs always require user authorization which also prevents silent automatic installs
* Slower relaunching and installation of updates on quit
* No support for [delta updates](/documentation/delta-updates) for more efficient updates
* No fallback for [rotating signing keys](/documentation#rotating-signing-keys) in case your signing keys need to change
* No support for generating updates easily using the `generate_appcast` tool

Applications that [install daemons](https://developer.apple.com/documentation/servicemanagement/smappservice) or [install system extensions](https://developer.apple.com/documentation/systemextensions/installing-system-extensions-and-drivers) do not need to distribute package installers.

As of Sparkle 2.7.3, installing package updates may not work in development builds of apps where Sparkle's helper tools are not usually re-signed. If this is the case, please [test Sparkle](/documentation#6-test-sparkle-out) either from a notarized version of your app, or from a version that was installed by your package installer.

### Bare Package Installation

Sparkle supports serving and signing flat `*.pkg` or `*.mpkg` packages directly without having to zip or archive them. This method requires users from old versions of your application to be using [Sparkle 1.26 or later](/documentation/upgrading/). If you have users running older versions of Sparkle, you can expedite migration by [switching to a new appcast](/documentation/publishing/#upgrading-to-newer-features), or use [Archived Package Installation](#archived-package-installation) until the majority of your users update.

This method is the recommended way of serving package based updates because it avoids redundant re-compression and metadata.

### Archived Package Installation

A package installation occurs when Sparkle finds a `*.pkg` or `*.mpkg` file in the root of the download archive (e.g, from within a `.zip`).

**Note**: For Sparkle 2, you must also add `sparkle:installationType="package"` to your appcast item's `enclosure` for updating archived packages.
