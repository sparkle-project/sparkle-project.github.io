---
layout: documentation
id: documentation
title: Package Updates
---
## Package Updates

Package installation allows Sparkle to update your application by downloading and installing a package, `pkg`, or multi-package, `mpkg` usually without user interaction except for asking for an administrator password.

### Automatic Archived Installation

An automatic archived installation occurs when Sparkle finds a `*.pkg` or `*.mpkg` file in the root of the download archive. [Older versions](/documentation/upgrading/) of Sparkle required the filename to be `*.sparkle_guided.pkg` to perform an automatic installation, so you may want to keep that name until majority of your users update to your application with Sparkle 1.16 or later.

**Note**: For Sparkle 2.0 (Beta), you must add `sparkle:installationType="package"` to your appcast item for updating automatic packages that are archived.

### Automatic Bare Installation

Sparkle [1.26 or later](/documentation/upgrading/) supports serving and signing flat `*.pkg` or `*.mpkg` packages directly without having to zip or archive them. You will want to keep archiving them however until majority of your users update to your application with a version of Sparkle that supports this.

### Interactive Archived UI Installation

An interactive installation occurs when Sparkle finds a `*.sparkle_interactive.pkg` or `*.sparkle_interactive.mpkg` file in the root of the download archive.

The package will be installed using macOS's built-in GUI installer. The installation will require user to manually click through the steps, so we don't recommend this type of installation. You must also archive your package update to get this behavior. This type of installation is deprecated in Sparkle 2 and may be removed one day.

**Note**: For Sparkle 2.0 (Beta), you must add `sparkle:installationType="interactive-package"` to your appcast item for updating interactive packages.
