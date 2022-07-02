---
layout: documentation
id: documentation
title: sparkle-cli
---

## sparkle-cli

Sparkle 2 includes a command line utility that can update Sparkle-based applications and bundles. This tool is a [thin wrapper around using Sparkle's framework](https://github.com/sparkle-project/Sparkle/tree/2.x/sparkle-cli) to update external bundles.

### Usage

```
./sparkle.app/Contents/MacOS/sparkle

Usage: ./sparkle.app/Contents/MacOS/sparkle bundle [--application app-path] [--check-immediately] [--probe] [--channels chan1,chan2,…] [--feed-url feed-url] [--user-agent-name display-name] [--grant-automatic-checks] [--send-profile] [--defer-install] [--interactive] [--allow-major-upgrades] [--verbose]
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
  authorization. Alternatively, this tool can be run as root under an active user login
  session, which will not require (and disallow) interaction.

  If --defer-install is specified, this tool will exit leaving a spawned process
  for finishing the installation after the target application terminates.

  If update installation fails due to not having permission (e.g. from Gatekeeper) to replace the old bundle, an exit status of 8 is returned.
  Please specify --user-agent-name if you intend to use this tool in an automated way.
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
 --allow-major-upgrades
    Allows probing and installing major upgrades. Without passing this, an exit
    status of 2 is returned if a major upgrade is found.
 --channels
    List of allowed Sparkle channels to look for updates in. By default,
    only the default channel is used.
 --feed-url
    URL for appcast feed. This URL will be used for the feed instead of the one
    in the bundle's Info.plist or in the bundle's user defaults.
 --user-agent-name
    Display name that will be included as a part of the User-Agent string.
    We encourage setting this so developers know what is querying their feed.
    Otherwise, this value may be set and inferred automatically.
 --interactive
    Allows prompting the user for an authorization dialog prompt if the
    installer needs elevated privileges, or allows performing an interactive
    installer package. Without passing this, an exit status of 3 is returned
    if an update requires user interaction. An exit status of 5 is returned
    if the user cancels the authorization prompt.
 --grant-automatic-checks
    If update permission is requested, this enables automatic update checks.
    Note that this behavior may overwrite the user's defaults for the bundle.
    This option has no effect if --check-immediately is passed, or if the
    user has replied to this request already, or if the developer configured
    to skip it. Without passing this, an exit status of 6 is returned
    if permission is needed.
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

One example is I updated an application on my machine I knew was out of date by running:

```sh
./sparkle.app/Contents/MacOS/sparkle --check-immediately /Applications/Hex\ Fiend.app/
```

### Caveats

There are caveats for updating applications you do not own with sparkle-cli. For example an app may implement Sparkle's delegate methods for using a custom version comparator or feed URL, but sparkle-cli has no way of knowing to use these if they are not extractable externally.

On macOS 13 (Ventura) and later, users will need to approve external updaters like sparkle-cli to make modifications to update other developer's applications. If you use an external updater like sparkle-cli to update your own application, make sure the bundle you're updating and the updater is signed with the same Team ID.
