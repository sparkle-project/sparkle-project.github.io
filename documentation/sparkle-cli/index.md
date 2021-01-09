---
layout: documentation
id: documentation
title: sparkle-cli
---

## sparkle-cli

Sparkle 2.0 includes a command line utility that can update Sparkle-based applications and bundles.

### Usage

Check out its usage:

```
./sparkle.app/Contents/MacOS/sparkle

Usage: sparkle.app/Contents/MacOS/sparkle bundle [--application <app-path>] [--check-immediately] [--probe] [--grant-automatic-checks] [--send-profile] [--defer-install] [--interactive] [--verbose]
Description:
  Check if any new updates for a Sparkle supported bundle need to be installed.

  If any new updates need to be installed, the user application
  is terminated and the update is installed immediately unless --defer-install
  is specified. If the application was alive, then it will be relaunched after.

  To check if an update is available without installing, use --probe.

  if no updates are available now, or if the last update check was recently
  (unless --check-immediately is specified) then nothing is done.

  If update permission is requested and --grant-automatic-checks is not
  specified, then checking for updates is aborted.

  Unless --interactive is specified, this tool will not request for escalated
  authorization. Running as root is not supported.

  If --defer-install is specified, this tool will exit leaving a spawned process
  for finishing the installation after the target application terminates.
Options:
 --application
    Path to the application to watch for termination and to relaunch.
    If not provided, this is assumed to be the same as the bundle.
 --check-immediately
    Immediately checks for updates to install.
    Without this, updates are checked only when needed on a scheduled basis.
 --probe
    Probe for updates. Check if any updates are available but do not install.
    An exit status of 0 is returned if a new update is available.
 --feed-url
    URL for appcast feed. This URL will be used for the feed instead of the one
    in the bundle's Info.plist or in the bundle's user defaults.
 --interactive
    Allows prompting the user for an authorization dialog prompt if the
    installer needs elevated privileges, or allows performing an interactive
    installer package.
 --grant-automatic-checks
    If update permission is requested, this enables automatic update checks.
    Note that this behavior may overwrite the user's defaults for the bundle.
    This option has no effect if --check-immediately is passed, or if the
    user has replied to this request already, or if the developer configured
    to skip it.
 --send-profile
    Choose to send system profile information if update permission is requested.
    This option can only take effect if --grant-automatic-checks is passed.
 --defer-install
    Defer installation until after the application terminates on its own. The
    application will not be relaunched unless the installation is resumed later.
 --verbose
    Enable verbose logging.
```

### Example

For example, I updated an application on my machine I knew was out of date by running:

```
./sparkle.app/Contents/MacOS/sparkle --check-immediately /Applications/Hex\ Fiend.app/
```

### Caveats

There are caveats for updating applications you do not own with sparkle-cli. For example an app may implement Sparkle's delegate methods for using a custom version comparator or feed URL, but sparkle-cli has no way of knowing to use these if they are not extractable externally.
