---
layout: documentation
id: documentation
title: Package Updates
---
## Package Updates

Package installation allows Sparkle to update your application by downloading and installing a package, `pkg`, or multi-package, `mpkg` usually without user interaction except for asking for an administrator password.

Note package installation should only be used for apps with very custom installation needs that cannot be satisfied by distributing a regular app bundle. For Sparkle, the downsides of using package updates are:

* Installs always require user authorization which also prevents silent automatic installs
* Slower relaunching and installation of updates on quit
* No support for [delta updates](/documentation/delta-updates) for more efficient updates
* No fallback for [rotating signing keys](/documentation#rotating-signing-keys) in case the app's Developer ID changes
* No support for generating updates easily using the `generate_appcast` tool

Applications that [install daemons](https://developer.apple.com/documentation/servicemanagement/smappservice) or [install system extensions](https://developer.apple.com/documentation/systemextensions/installing_system_extensions_and_drivers) do not need to distribute package installers.

### Bare Package Installation

Sparkle supports serving and signing flat `*.pkg` or `*.mpkg` packages directly without having to zip or archive them. This method requires users from old versions of your application to be using [Sparkle 1.26 or later](/documentation/upgrading/). If you have users running older versions of Sparkle, you can expedite migration by [switching to a new appcast](/documentation/publishing/#upgrading-to-newer-features), or use [Archived Package Installation](#archived-package-installation) until the majority of your users update.

This method is the recommended way of serving package based updates because it avoids redundant re-compression and metadata.

### Archived Package Installation

A package installation occurs when Sparkle finds a `*.pkg` or `*.mpkg` file in the root of the download archive (e.g, from within a `.zip`).

**Note**: For Sparkle 2, you must also add `sparkle:installationType="package"` to your appcast item's `enclosure` for updating archived packages.

### Interactive Archived UI Installation

**Warning**: This type of installation is deprecated and may be removed one day. Please don't use it for future updates to your application.

An interactive installation occurs when Sparkle finds a `*.sparkle_interactive.pkg` or `*.sparkle_interactive.mpkg` file in the root of the download archive.

The package will be installed using macOS's built-in GUI installer. The installation will require user to manually click through the steps, so we don't recommend this type of installation. You must also archive your package update (e.g, in a `.zip`) to get this behavior.

**Note**: For Sparkle 2, you must also add `sparkle:installationType="interactive-package"` to your appcast item's `enclosure` for updating interactive packages.
