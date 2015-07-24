---
layout: documentation
id: documentation
title: Package Updates
---
## Package Updates

Guided Package Installation allows Sparkle to download and install a package, `pkg`, or multi-package, `mpkg`, without user interaction (other than asking for an administrator password).

### Deploying a Package

A guided installation occurs when Sparkle finds a `*.sparkle_guided.pkg` or `*.sparkle_guided.mpkg` in the root of the download.

The installer package is installed using OS X's built-in command line installer, `/usr/sbin/installer`. No installation interface is shown to the user.

A guided installation can be started by applications other than the application being replaced. This is particularly useful where helper applications or agents are used.
