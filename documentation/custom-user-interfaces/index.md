---
layout: documentation
id: documentation
title: Custom User Interfaces
---

## Custom User Interfaces

Sparkle's standard user interface handles many standard behaviors for you. However, some applications may want their own tailored user interface. Before switching to a custom UI for your updater, you should consider if your application will provide the updating functionality users want. For example, users appreciate viewing release notes when being shown a new update is available. Some users may also prefer to reduce update notifications/badges/alerts/custom UI indicators altogether by automatically downloading and installing updates in the future. You may also want to check [gentle update reminders](/documentation/gentle-reminders) for changing how Sparkle's standard update alert can be presented.

### SPUUserDriver

[SPUUserDriver](/documentation/api-reference/Protocols/SPUUserDriver.html) is a protocol that drives the user interface of [SPUUpdater](/documentation/api-reference/Classes/SPUUpdater.html).

To create your own user interface, you can create your own class which conforms to [SPUUserDriver](/documentation/api-reference/Protocols/SPUUserDriver.html). Please read the API documentation for all the methods you implement. Then you can [instantiate a SPUUpdater](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(im)initWithHostBundle:applicationBundle:userDriver:delegate:) with your user driver instead of creating the typical [SPUStandardUpdaterController](/documentation/api-reference/Classes/SPUStandardUpdaterController.html).

Reference classes in Sparkle's repository that implement `SPUUserDriver` are:

* [SPUStandardUserDriver](https://github.com/sparkle-project/Sparkle/blob/2.x/Sparkle/SPUStandardUserDriver.m): Sparkle's standard user interface
* [SUPopUpTitlebarUserDriver](https://github.com/sparkle-project/Sparkle/blob/2.x/TestApplication/SUPopUpTitlebarUserDriver.m): Alternative user interface to the Sparkle Test App
* [SUCommandLineUserDriver](https://github.com/sparkle-project/Sparkle/blob/2.x/sparkle-cli/SPUCommandLineUserDriver.m): User interface to the [sparkle-cli](/documentation/sparkle-cli) tool

Futher sections below highlight scenarios you will need to consider when implementing a custom UI.

### Update Check Permission

By default, Sparkle asks users for their permission to check for new updates automatically on your application's second launch (along with options like if updates should also be downloaded and installed automatically). To add support for this permission request, a user driver needs to implement [-[SPUUserDriver showUpdatePermissionRequest:reply:]](/documentation/api-reference/Protocols/SPUUserDriver.html#/c:objc(pl)SPUUserDriver(im)showUpdatePermissionRequest:reply:).

To test this permision request, you can reset the response by running:

```sh
# Automatic checks for updates
defaults delete my-app-bundle-id SUEnableAutomaticChecks
# Automatic downloading/installing of updates
defaults delete my-app-bundle-id SUAutomaticallyUpdate
# Sending anonymous system profile information
defaults delete my-app-bundle-id SUSendProfileInfo
```

Applications that don't want to ask the user permission for update checking may opt-out by specifying [SUEnableAutomaticChecks in Info.plist](/documentation/customization/). In this case, this user driver method will not be called.

### Authorization

Custom user interfaces need to handle updates that require user authorization to install them. Before installing an update, users on some systems may be requested for an administrator username/password to authorize the installation. While user authorization is always required for [package updates](/documentation/package-updates), authorization can also be required for regular updates on certain systems (e.g. updating a bundle in `/Applications` on standard user accounts or managed systems). To test authorized installs, you can change the ownership of a bundle to `root` before checking for updates:

```sh
sudo chown root path-to-app-bundle
```

This implies your user interface needs to let the user be in control of installing an update once [an update is found](/documentation/api-reference/Protocols/SPUUserDriver.html#/c:objc(pl)SPUUserDriver(im)showUpdateFoundWithAppcastItem:state:reply:) so they do not receive an unexpected authorization prompt. If you want as much as the downloading and installing of an update to be done in the background before a user's input or presenting an update is needed, you can opt into automatic updates via [SUAutomaticallyUpdate](/documentation/customization/).

### Update States

When a new update is found the updater can be in one of several different stages. The most regular stage is when an update has not been downloaded yet. The second most typical stage is when the update has already begun installing in the background (due to automatic updates via [SUAutomaticallyUpdate](/documentation/customization/)); in this case, the application may be immediately relaunched if the user chooses to install the update, or just installed on quit without any additional input. Another stage is when the update has been automatically downloaded but not begun installing yet, which is typically for automatic updates when the updater needs to request for user authorization.

Lastly, custom user interfaces should be able to handle [informational only updates](/documentation/publishing/#downloading-from-a-web-site) as a safety net in case updates cannot be served normally.

Please read the documentation for [-[SPUUserDriver showUpdateFoundWithAppcastItem:state:reply:]](/documentation/api-reference/Protocols/SPUUserDriver.html#/c:objc(pl)SPUUserDriver(im)showUpdateFoundWithAppcastItem:state:reply:) for detailed information on different update states.

### Update Back in Focus

After an update has been shown, a user can check for updates again to bring the update UI back in frontmost focus. To support this you need to implement [-[SPUUserDriver showUpdateInFocus]](/documentation/api-reference/Protocols/SPUUserDriver.html#/c:objc(pl)SPUUserDriver(im)showUpdateInFocus). This method used to be required but is now optional since Sparkle 2.8. Don't implement this method if bringing an update back in focus does not make sense for your user interface (e.g. update UI is always in frontmost focus).

