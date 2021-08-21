---
layout: documentation
id: documentation
title: Setting up Sparkle programmatically
---

## Create an updater programmatically

Here is an example of creating an updater in Sparkle 2 (beta) with Cocoa programmatically. It hooks up a menu item's target/action for checking for updates.

```swift
import Cocoa
import Sparkle

@NSApplicationMain
@objc class AppDelegate: NSObject, NSApplicationDelegate {
    @IBOutlet var checkForUpdatesMenuItem: NSMenuItem! // Hooked up in Interface Builder

    let updaterController = SPUStandardUpdaterController(startingUpdater: false, updaterDelegate: nil, userDriverDelegate: nil)

    func applicationDidFinishLaunching(_ notification: Notification) {
        checkForUpdatesMenuItem.target = updaterController
        checkForUpdatesMenuItem.action = #selector(SPUStandardUpdaterController.checkForUpdates(_:))

        // SPUStandardUpdaterController was instantiated by not starting the updater automatically
        updaterController.startUpdater()
    }
}
```

In Sparkle 2 you can also choose to instantiate and use `SPUUpdater` directly instead of the `SPUStandardUpdaterController` wrapper if you need more control over your user interface or what bundle to update.

If you use another UI toolkit, these are the relevant APIs in Sparkle 2 for checking for updates and handling menu item validation:

* `-[SPUStandardUpdaterController startUpdater]` or `-[SPUUpdater startUpdater:]` for starting the updater (unless it is specified to be automatically started)
* `-[SPUStandardUpdaterController checkForUpdates:]` or `-[SPUUpdater checkForUpdates]` for checking for updates
* `-[SPUUpdater canCheckForUpdates]` for menu item validation

If you are using Sparkle 1, you will need to use these APIs:

* `+[SUUpdater sharedUpdater]` for creating and starting the updater automatically
* `-[SUUpdater checkForUpdates:]` for checking for updates
* `-[SUUpdater validateMenuItem:]` for menu item validation

Check [Sparkle's APIs](/documentation/customization) for more information.
