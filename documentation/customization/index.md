---
layout: documentation
id: documentation
title: Customizing Sparkle
---
## Customizing Sparkle

Besides the `SUFeedURL` and `SUPublicEDKey` options, Sparkle will choose defaults for your app's update behavior. However you can change these settings if you want different behavior.

For example:

* If you do not want Sparkle to prompt the user (on second app launch) asking for permission if your app can check for updates automatically in the background, set `SUEnableAutomaticChecks` in your app's Info.plist.
* If you want Sparkle to automatically download and install updates silently in the background instead of notifying users of updates every time by default, enable `SUAutomaticallyUpdate` in your app's Info.plist.

Some of the Info.plist settings below are also paired with an updater API to change the settings at runtime, like `SUEnableAutomaticChecks` with [automaticallyChecksForUpdates](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(py)automaticallyChecksForUpdates). The Info.plist settings are meant for default configuration, while the runtime APIs are in response to user setting changes. Please do not use the runtime APIs for setting initial default behavior. Check how to [add update settings](/documentation/preferences-ui/) for examples on changing user settings.

### General Settings

{:.table .table-bordered}
| Key | Type | Value |
| --- | ---- | ----- | ------- |
| `SUFeedURL` | String | The URL of your appcast, e.g. `https://example.com/appcast.xml`. It's recommended to set it in Info.plist, even if you change it later programmatically. |
| `SUPublicEDKey` | String | The base64-encoded public EdDSA key. Use Sparkle's `generate_keys` tool to get it. |
| `SUEnableAutomaticChecks` | Boolean | By default when this is not set, automatic checking for updates is initially disabled but users will be prompted for permission for Sparkle to check for updates automatically on second launch. If permission is granted, automatic checks for updates then becomes enabled. As of Sparkle 2.4 or later, the user will also have an option to opt in or out of automatic downloading of updates; the default state is based on the `SUAutomaticallyUpdate` setting.<br/>Setting this to `YES` enables automatic checking for updates (but not installation) by default, without asking your users for permission first.<br>Setting this to `NO` disables automatic checking for updates by default, also without asking your users for permission.<br/>This default property can later be overridden by setting [automaticallyChecksForUpdates](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(py)automaticallyChecksForUpdates) in response to user setting changes. |
| `SUScheduledCheckInterval` | Number | The number of seconds between automatic update checks. The default is `86400` (1 day).<br /><br />**Note:** this has a minimum bound of 1 hour in order to keep you from accidentally overloading your servers. |
| `SUAutomaticallyUpdate` | Boolean | Default: `NO`. Enables automatic download and installation of updates by default. If set to `YES`, Sparkle will attempt to download and install new updates silently in the background. Updates may be downloaded but not installed automatically if authorization is required. If the application hasn't quit for `SUScheduledImpatientCheckInterval` the user will be presented with installing the downloaded update (unless the updater delegate [overrides this](/documentation/api-reference/Protocols/SPUUpdaterDelegate.html#/c:objc(pl)SPUUpdaterDelegate(im)updater:willInstallUpdateOnQuit:immediateInstallationBlock:)).<br/>This default property can later be overridden by setting [automaticallyDownloadsUpdates](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(py)automaticallyDownloadsUpdates) in response to user setting changes (for example, the standard update alert has a checkbox for changing this).
| `SUScheduledImpatientCheckInterval` | Number | The number of seconds between update checks after an update has been set to install in the background via `SUAutomaticallyUpdate`. The default is `604800` (1 week). If the user hasn't quit their app within this interval yet, they may be notified of the update that's ready to be installed. Note this interval must be bigger than `SUScheduledCheckInterval`. This setting is configurable since Sparkle 2.9 (beta). |
| `SUAllowsAutomaticUpdates` | Boolean | By default, Sparkle automatically presents your users with the *option* to allow to automatically download and install any available updates if automatic checking of updates is enabled.<br/>Set this to `NO` to disallow automatic updates and require manual installation every time.<br/>Set this to `YES` to always allow automatic updates even if automatic checking of updates is disabled. |
| `SUEnableSystemProfiling` | Boolean | Default: `NO`. Enables anonymous system profiling. See [System Profiling](/documentation/system-profiling) for more. |
| `SUShowReleaseNotes` | Boolean | Default: `YES`. Set this to `NO` to hide release notes display from the update alert. |
| `SUBundleName` | String | Optional alternative bundle display name. For example, if your bundle name already has a version number appended to it, setting this may help smooth out certain messages, e.g. "MyApp 3 4.0 is now available" vs "MyApp 4.0 is now available". |
| `SUDefaultsDomain` | String | Optional alternative `NSUserDefaults` domain name if you don't want to use the standard user defaults, for example when accessing preferences from an App Group suite. |
| `SURelaunchHostBundle` | Boolean | Default: `NO`. For plug-ins in Sparkle 2, set this to `YES` to re-launch the host targetted bundle instead of the application bundle. For example, this can be used to re-open a System Settings prefpane after an update has been installed.

### Security Settings

{:.table .table-bordered}
| Key | Type | Value |
| --- | ---- | ----- | ------- |
| `SUVerifyUpdateBeforeExtraction` | Boolean | Default: `NO`. Set this to `YES` to force verification of updates before Sparkle extracts the downloaded update. Use this setting if you want stronger update validation and you aren't likely to lose access to your private EdDSA key (typically stored inside the macOS Keychain). EdDSA signing is required to use this setting. [Changing EdDSA keys](/documentation/#rotating-signing-keys) is still possible by using Apple Developer ID code signed disk images as a fallback. This setting is available since Sparkle 2.7.3. |
| `SURequireSignedFeed` | Boolean | Default: `NO`. Set this to `YES` to make Sparkle validate appcasts and release notes are signed. Use this setting if you want secure validation of the update information presented to the user, and you aren't likely to lose access to your private EdDSA key (typically stored inside the macOS Keychain). This setting is available since Sparkle 2.9 (beta), and also requires enabling `SUVerifyUpdateBeforeExtraction` as a prerequisite.
| `SUSignedFeedFailureExpirationInterval` | Number | The number of seconds it takes for a feed signing validation failure to expire. The default is `1728000` (20 days), and a special value of `0` disables feed signing failures from expiring. If Sparkle hasn't successfully validated a feed for this amount of time after the first signing failure, it will finally present new updates to users. This setting is a failsafe in case you lose access to your EdDSA signing keys, but can still serve updates through [rotating keys](/documentation/#rotating-signing-keys). In this scenario, release notes are not shown and informational only updates are not supported. `SURequireSignedFeed` must be enabled for this setting to take effect.
| `SUAllowedURLSchemes` | Array of Strings | An array of custom URL schemes allowed to be clicked from Sparkle's release notes view. By default, Sparkle only allows clicks to links that have a safe known URL scheme (like `https`). This setting is available in Sparkle 2.5 and later.
| `SUEnableJavaScript` | Boolean | Default: `NO`. Set this to `YES` if you want to allow JavaScript in your release notes. |

### Sandboxing Settings

Here are the Info.plist settings relevant to use for Sandboxed applications using Sparkle 2. Applications that are not sandboxed should not customize any of these settings. Please visit the [Sandboxing guide](/documentation/sandboxing) for more detailed information.

{:.table .table-bordered}
| Key | Type | Value |
| --- | ---- | ----- | ------- |
| `SUEnableInstallerLauncherService` | Boolean | Default: `NO`. Set this to `YES` to enable using the Installer Launcher XPC Service, which is required for all Sandboxed applications. Do not enable this XPC Service if your application is not sandboxed (sandboxed applications must have the `com.apple.security.app-sandbox` entitlement). |
| `SUEnableDownloaderService` | Boolean | Default: `NO`. Set this to `YES` to enable using the Downloader XPC Service. Do not enable this XPC Service if your sandboxed application already requests the `com.apple.security.network.client` (Outgoing Network Connections) entitlement. |
| `SUEnableInstallerConnectionService` | Boolean | Default: `NO`. Set this to `YES` to enable using the Installer Connection Service. This service is usually not needed due to being able to [add a mach name lookup exception entitlement](/documentation/sandboxing#installer-connection--status-services) instead. This service is not included in Sparkle's framework distribution by default. |
| `SUEnableInstallerStatusService` | Boolean | Default: `NO`. Set this to `YES` to enable using the Installer Status Service. This service is usually not needed due to being able to [add a mach name lookup exception entitlement](/documentation/sandboxing#installer-connection--status-services) instead. This service is not included in Sparkle's framework distribution by default. |

### Sparkle APIs

Please visit the [Sparkle 2 API Reference](/documentation/api-reference) for full documentation on class and delegate APIs.
