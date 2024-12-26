---
layout: documentation
id: documentation
title: Customizing Sparkle
---
## Customizing Sparkle

Here are the main routes by which you can bend Sparkle's behavior to your will:

### Info.plist Settings

{:.table .table-bordered}
| Key | Type | Value |
| --- | ---- | ----- | ------- |
| `SUFeedURL` | String | The URL of your appcast, e.g. `https://example.com/appcast.xml`. It's recommended to always set it in Info.plist, even if you change it later programmatically. |
| `SUPublicEDKey` | String | The base64-encoded public EdDSA key. Use Sparkle's `generate_keys` tool to get it. |
| `SUEnableAutomaticChecks` | Boolean | By default when this is not set, automatic checking for updates is initially disabled but users will be prompted for permission for Sparkle to check for updates automatically on second launch. If permission is granted, automatic checks for updates then becomes enabled. As of Sparkle 2.4 or later, the user will also have an option to opt into or out of automatic downloading of updates; the default state is based on the `SUAutomaticallyUpdate` setting.<br/>Setting this to `YES` enables automatic checking for updates (but not installation) by default, without asking your users for permission first.<br>Setting this to `NO` disables automatic checking for updates by default, also without asking your users for permission.<br/>This default property can later be overridden by setting [automaticallyChecksForUpdates](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(py)automaticallyChecksForUpdates) in response to user setting changes. |
| `SUEnableSystemProfiling` | Boolean | Default: `NO`. Enables anonymous system profiling. See [System Profiling](/documentation/system-profiling) for more. |
| `SUScheduledCheckInterval` | Number | The number of seconds between updates. The default is `86400` (1 day). Setting to 0 disables updates.<br /><br />**Note:** this has a minimum bound of 1 hour in order to keep you from accidentally overloading your servers. |
| `SUAllowsAutomaticUpdates` | Boolean | By default, Sparkle automatically presents your users with the *option* to allow to automatically download and install any available updates if automatic checking of updates is enabled.<br/>Set this to `NO` to disallow automatic updates and require manual installation every time.<br/>Set this to `YES` to always allow automatic updates even if automatic checking of updates is disabled. |
| `SUAutomaticallyUpdate` | Boolean | Default: `NO`. Enables automatic download and installation of updates by default. If set to `YES`, Sparkle will attempt to download and install new updates silently in the background. In Sparkle 1, updates won't be opted into this if users need to provide authorization. In Sparkle 2, updates will be downloaded but not installed automatically if authorization is required. In all versions of Sparkle, if the application hasn't quit for 1 week the user will be presented with installing the downloaded update (unless the updater delegate overrides this).<br/>This default property can later be overridden by setting [automaticallyDownloadsUpdates](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(py)automaticallyDownloadsUpdates) in response to user setting changes.
| `SUVerifyUpdateBeforeExtraction` | Boolean | Default: `NO`. Set this to `YES` to force verification of updates before Sparkle extracts the downloaded update. Use this setting if you want stronger update validation and you aren't likely to lose access to your private EdDSA key (typically stored inside the macOS Keychain). EdDSA signing is required to use this setting. [Key rotation](/documentation/#rotating-signing-keys) is still possible by using Apple Developer ID code signed disk images as fallback. This setting is available in Sparkle 2.7 (beta) and later. |
| `SUShowReleaseNotes` | Boolean | Default: `YES`. Set this to `NO` to hide release notes display from the update alert. |
| `SUAllowedURLSchemes` | Array of Strings | An array of custom URL schemes allowed to be clicked from Sparkle's release notes view. By default, Sparkle only allows clicks to links that have a safe known URL scheme (like `https`). This setting is available in Sparkle 2.5 and later.
| `SUBundleName` | String | Optional alternative bundle display name. For example, if your bundle name already has a version number appended to it, setting this may help smooth out certain messages, e.g. "MyApp 3 4.0 is now available" vs "MyApp 4.0 is now available". |
| `SUDefaultsDomain` | String | Optional alternative `NSUserDefaults` domain name if you don't want to use the standard user defaults, for example when accessing preferences from an App Group suite. |
| `SUEnableJavaScript` | Boolean | Default: `NO`. Set this to `YES` if you want to allow JavaScript in your release notes. |
| `SURelaunchHostBundle` | Boolean | Default: `NO`. For plug-ins in Sparkle 2, set this to `YES` to re-launch the host targetted bundle instead of the application bundle. For example, this can be used to re-open a System Settings prefpane after an update has been installed.

### Sandboxing Settings

Here are the Info.plist settings relevant to use for Sandboxed applications using Sparkle 2. Applications that are not sandboxed should not customize any of these settings. Please visit the [Sandboxing guide](/documentation/sandboxing) for more detailed information.

{:.table .table-bordered}
| Key | Type | Value |
| --- | ---- | ----- | ------- |
| `SUEnableInstallerLauncherService` | Boolean | Default: `NO`. Set this to `YES` to enable using the Installer Launcher XPC Service, which is required for all Sandboxed applications. Do not enable this XPC Service if your application is not sandboxed (sandboxed applications must have the `com.apple.security.app-sandbox` entitlement). |
| `SUEnableDownloaderService` | Boolean | Default: `NO`. Set this to `YES` to enable using the Downloader XPC Service. Do not enable this XPC Service if your sandboxed application already requests the `com.apple.security.network.client` (Outgoing Network Connections) entitlement. |
| `SUEnableInstallerConnectionService` | Boolean | Default: `NO`. Set this to `YES` to enable using the Installer Connection Service. This service is usually not needed due to being able to [add a mach name lookup exception entitlement](/documentation/sandboxing#installer-connection--status-services) instead. This service is not included in Sparkle's framework distribution by default. |
| `SUEnableInstallerStatusService` | Boolean | Default: `NO`. Set this to `YES` to enable using the Installer Status Service. This service is usually not needed due to being able to [add a mach name lookup exception entitlement](/documentation/sandboxing#installer-connection--status-services) instead. This service is not included in Sparkle's framework distribution by default. |

### Sparkle 2 APIs

Please visit the [Sparkle 2 API Reference](/documentation/api-reference).

### Sparkle 1 APIs

#### Calls to SUUpdater

The `SUUpdater` object is the main controller for the updating system in your app. There is a singleton instance of this class for each bundle being updated. If you're trying to update the running .app, you can retrieve the appropriate `SUUpdater` by calling `[SUUpdater sharedUpdater]`. If you're trying to update some [other bundle](/documentation/bundles/), you can use `[SUUpdater updaterForBundle:(NSBundle *)myBundle]`.

Once you have the `SUUpdater` instance, there are a few interesting accessors you could use. Please use them only if you need dynamic behavior (e.g. user preferences). Do not use these functions to set default configuration. Use Info.plist keys to set default configuration instead.

```objc
- (void)setAutomaticallyChecksForUpdates:(BOOL)automaticallyChecks;
- (BOOL)automaticallyChecksForUpdates;

- (void)setUpdateCheckInterval:(NSTimeInterval)interval;
- (NSTimeInterval)updateCheckInterval;

- (void)setFeedURL:(NSURL *)feedURL;
- (NSURL *)feedURL;

- (void)setSendsSystemProfile:(BOOL)sendsSystemProfile;
- (BOOL)sendsSystemProfile;

- (void)setAutomaticallyDownloadsUpdates:(BOOL)automaticallyDownloadsUpdates;
- (BOOL)automaticallyDownloadsUpdates;
```

There is a risk of race conditions. If you want to make sure these settings are changed before the first automatic update check, you should do this in the `NSApplication` delegate method `-applicationWillFinishLaunching:`. For a non-app bundle you should then make the changes immediately after you first create the `SUUpdater` instance in your code.

A few more methods of interest:

```objc
// This IBAction is meant for a main menu item. Hook up any menu item to this action,
// and Sparkle will check for updates and report back its findings through UI.
- (IBAction)checkForUpdates:sender;

// This kicks off an update meant to be programmatically initiated. That is,
// it will display no UI unless it actually finds an update, in which case it
// proceeds as usual. If the automated downloading is turned on, however,
// this will invoke that behavior, and if an update is found, it will be
// downloaded and prepped for installation.
//
// You do not need to call this. Sparkle calls it automatically according to
// the update schedule.
- (void)checkForUpdatesInBackground;

// This begins a "probing" check for updates which will not actually offer to
// update to that version. The delegate methods, though, (up to updater:didFindValidUpdate:
// and updaterDidNotFindUpdate:), are called, so you can use that information in your UI.
// Essentially, you can use this to UI-lessly determine if there's an update.
- (void)checkForUpdateInformation;

// Date of last update check. Returns nil if no check has been performed.
- (NSDate *)lastUpdateCheckDate;

// Call this to appropriately schedule or cancel the update checking timer according
// to the preferences for time interval and automatic checks. If this SUUpdater instance
// was not present during the application's launch, you must call this method to start
// the update cycle explicitly.
- (void)resetUpdateCycle;

- (BOOL)updateInProgress;

- (void)setDelegate:(id)delegate; // See below for more information on the delegate.
- delegate;
```

#### SUUpdater delegate methods

You can control the SUUpdater's behavior a little more closely by providing it with a delegate. Here are the delegate methods you might implement:

```objc
// Use this to override the default behavior for Sparkle prompting the
// user about automatic update checks. You could use this to make Sparkle
// prompt for permission on the first launch instead of the second.
- (BOOL)updaterShouldPromptForPermissionToCheckForUpdates:(SUUpdater *)bundle;

- (void)updater:(SUUpdater *)updater didFinishLoadingAppcast:(SUAppcast *)appcast;

// If you're using special logic or extensions in your appcast, implement
// this to use your own logic for finding a valid update, if any, in the given appcast.
- (SUAppcastItem *)bestValidUpdateInAppcast:(SUAppcast *)appcast
                   forUpdater:(SUUpdater *)bundle;

- (void)updater:(SUUpdater *)updater didFindValidUpdate:(SUAppcastItem *)update;
- (void)updaterDidNotFindUpdate:(SUUpdater *)update;

// Sent immediately before installing the specified update.
- (void)updater:(SUUpdater *)updater willInstallUpdate:(SUAppcastItem *)update;

// Return YES to delay the relaunch until you do some processing.
// Invoke the provided NSInvocation to continue the relaunch.
- (BOOL)updater:(SUUpdater *)updater
        shouldPostponeRelaunchForUpdate:(SUAppcastItem *)update
        untilInvoking:(NSInvocation *)invocation;

// Called immediately before relaunching.
- (void)updaterWillRelaunchApplication:(SUUpdater *)updater;

// Called if the application has been relaunched from an update
- (void)updaterDidRelaunchApplication:(SUUpdater *)updater;

// This method allows you to provide a custom version comparator.
// If you don't implement this method or return nil, the standard version
// comparator will be used. See SUVersionComparisonProtocol.h for more.
- (id <SUVersionComparison>)versionComparatorForUpdater:(SUUpdater *)updater;

// Returns the path which is used to relaunch the client after the update
// is installed. By default, the path of the host bundle.
- (NSString *)pathToRelaunchForUpdater:(SUUpdater *)updater;

// This method allows you to add extra parameters to the appcast URL,
// potentially based on whether or not Sparkle will also be sending along
// the system profile. This method should return an array of dictionaries
// with keys: "key", "value", "displayKey", "displayValue", the latter two
// being human-readable variants of the former two.
- (NSArray *)feedParametersForUpdater:(SUUpdater *)updater
             sendingSystemProfile:(BOOL)sendingProfile;
```

#### Other options

If these methods aren't enough to do what you need, you're going to have to dig into Sparkle's code. You might start by creating a different update driver: check out SUBasicUpdateDriver.h to get an idea.
