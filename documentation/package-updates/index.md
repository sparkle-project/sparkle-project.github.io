---
layout: documentation
id: documentation
title: Package Updates
---
## Package Updates

Package installation allows Sparkle to update your application by downloading and installing a package, `pkg`, or multi-package, `mpkg` usually without user interaction except for asking for an administrator password.

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
