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
| `SUEnableAutomaticChecks` | Boolean | Setting to `YES` (recommended) enables checking for updates (but not installation) by default, without asking your users for permission first. <br>By default, if it's not set to any value, users will be prompted for permission before the first update check. <br>Setting to `NO` disables update checks, but can be overridden by a call to `SUUpdater`'s `setAutomaticallyChecksForUpdates:` |
| `SUPublicEDKey` | String | The base64-encoded public EdDSA key. Use Sparkle's `generate_keys` tool to get it. |
| `SUEnableSystemProfiling` | Boolean | Default: `NO`. Enables anonymous system profiling. See [System Profiling](/documentation/system-profiling) for more. |
| `SUScheduledCheckInterval` | Number | The number of seconds between updates. The default is `86400` (1 day). Setting to 0 disables updates. <br /><br />**Note:** this has a minimum bound of 1 hour in order to keep you from accidentally overloading your servers. |
| `SUAllowsAutomaticUpdates` | Boolean | Default: `YES`. Sparkle presents your users with the *option* to allow to automatically download and install any available updates. Set this to `NO` to disallow automatic updates and require manual installation every time. |
| `SUAutomaticallyUpdate` | Boolean | Default: `NO`. Enables automatic download and installation of updates. If set to `YES`, users will not be informed about updates, and updates will be silently installed when the app quits.
| `SUShowReleaseNotes` | Boolean | Default: `YES`. Set this to `NO` to hide release notes display from the update alert. |
| `SUBundleName` | String | Optional alternative bundle display name. For example, if your bundle name already has a version number appended to it, setting this may help smooth out certain messages, e.g. "MyApp 3 4.0 is now available" vs "MyApp 4.0 is now available". |
| `SUDefaultsDomain` | String | Optional alternative `NSUserDefaults` domain name if you don't want to use the standard user defaults, for example when accessing preferences from an App Group suite. |

### Calls to SUUpdater

The `SUUpdater` object is the main controller for the updating system in your app. There is a singleton instance of this class for each bundle being updated. If you're trying to update the running .app, you can retrieve the appropriate `SUUpdater` by calling `[SUUpdater sharedUpdater]`. If you're trying to update some [other bundle](/documentation/bundles/), you can use `[SUUpdater updaterForBundle:(NSBundle *)myBundle]`.

Once you have the `SUUpdater` instance, there are a few interesting accessors you could use. Please use them only if you need dynamic behavior (e.g. user preferences). Do not use these functions to set default configuration. Use Info.plist keys to set default configuration instead.

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

There is a risk of race conditions. If you want to make sure these settings are changed before the first automatic update check, you should do this in the `NSApplication` delegate method `-applicationWillFinishLaunching:`. For a non-app bundle you should then make the changes immediately after you first create the `SUUpdater` instance in your code.

A few more methods of interest:

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

### SUUpdater delegate methods

You can control the SUUpdater's behavior a little more closely by providing it with a delegate. Here are the delegate methods you might implement:

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

### Other options

If these methods aren't enough to do what you need, you're going to have to dig into Sparkle's code. You might start by creating a different update driver: check out SUBasicUpdateDriver.h to get an idea.
