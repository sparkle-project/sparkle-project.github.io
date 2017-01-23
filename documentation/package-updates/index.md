---
layout: documentation
id: documentation
title: Package Updates
---
## Package Updates

Guided Package Installation allows Sparkle to download and install a package, `pkg`, or multi-package, `mpkg`, without user interaction (other than asking for an administrator password).

### Automatic installation

A guided installation occurs when Sparkle finds a `*.pkg` or `*.mpkg` file in the root of the download archive. [Older versions](/documentation/upgrading/) of Sparkle required the filename to be `*.sparkle_guided.pkg`, so you may want to keep that name until majority if your users updates to app with Sparkle 1.16 or later.

The installer package is installed using macOS's built-in command line installer, `/usr/sbin/installer`. No installation interface is shown to the user.

A guided installation can be started by applications other than the application being replaced. This is particularly useful where helper applications or agents are used.

### Interactive GUI Installer

An interactive installation occurs when Sparkle finds a `*.sparkle_interactive.pkg` or `*.sparkle_interactive.mpkg` file in the root of the download archive.

The package will be installed using macOS's built-in GUI installer. The installation will require user to manually click through the steps, so we don't recommend this type of installation.
