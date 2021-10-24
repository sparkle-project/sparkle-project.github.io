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
    // Hook up menu item outlet in Interface Builder
    @IBOutlet var checkForUpdatesMenuItem: NSMenuItem!
    
    let updaterController: SPUStandardUpdaterController
    
    override init() {
        // If you want to start the updater manually, pass false to startingUpdater and call .startUpdater() later
        updaterController = SPUStandardUpdaterController(startingUpdater: true, updaterDelegate: nil, userDriverDelegate: nil)
    }
    
    func applicationDidFinishLaunching(_ notification: Notification) {
        checkForUpdatesMenuItem.target = updaterController
        checkForUpdatesMenuItem.action = #selector(SPUStandardUpdaterController.checkForUpdates(_:))
    }
}
```

In Sparkle 2 you can also choose to instantiate and use [SPUUpdater](/documentation/api-reference/Classes/SPUUpdater.html) directly instead of the [SPUStandardUpdaterController](/documentation/api-reference/Classes/SPUStandardUpdaterController.html) wrapper if you need more control over your user interface or what bundle to update.

If you use another UI toolkit, these are the relevant APIs in Sparkle 2 for checking for updates and handling menu item validation:

* [-[SPUStandardUpdaterController startUpdater]](/documentation/api-reference/Classes/SPUStandardUpdaterController.html#/c:objc(cs)SPUStandardUpdaterController(im)startUpdater) or [-[SPUUpdater startUpdater:]](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(im)startUpdater:) for starting the updater (unless it is specified to be automatically started)
* [-[SPUStandardUpdaterController checkForUpdates:]](/documentation/api-reference/Classes/SPUStandardUpdaterController.html#/c:objc(cs)SPUStandardUpdaterController(im)checkForUpdates:) or [-[SPUUpdater checkForUpdates]](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(im)checkForUpdates) for checking for updates
* [-[SPUUpdater canCheckForUpdates]](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(py)canCheckForUpdates) for menu item validation. This property is also KVO compliant.

If you are using Sparkle 1, you will need to use these APIs:

* `+[SUUpdater sharedUpdater]` for creating and starting the updater automatically
* `-[SUUpdater checkForUpdates:]` for checking for updates
* `-[SUUpdater validateMenuItem:]` for menu item validation
