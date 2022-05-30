---
layout: documentation
id: documentation
title: Gentle Update Reminders
---

**Note**: This article is for Sparkle 2.2, which is currently unreleased.

## Scheduled Update Checks

By default, when your application launches for the first time Sparkle is not granted permission to make any update checks. On the second launch of your application, Sparkle prompts the user asking if it is allowed to check for new updates automatically. This is defined by the [`SUEnableAutomaticChecks`](/documentation/customization) setting.

Once Sparkle is granted permission, it is programmed to schedule update checks on a regular basis defined by the [`SUScheduledCheckInterval`](/documentation/customization/) setting.

Letting Sparkle handle checking for updates automatically provides a couple benefits:
* It ensures your server is not polled too often (for when an app is re-launched frequently)
* It ensures update checks trigger often enough (for when your app has not quit for a long time)

Keeping your users up to date is important, however users do not want to have their focus stolen from new update alerts at inopportune times.

As of Sparkle 2.2, Sparkle prioritizes showing scheduled update alerts for regular (non-backgrounded) applications at opportune times when:
* The user just launched your app or just granted your app permission to check for updates automatically
* The user's system has been idle for some time while your app is active
* The user comes back to your application from another application (preferred over showing an alert when a user is actively using your app)

Note even for updates downloaded automatically and silently in the background, Sparkle may show an update alert to the user if the application hasn't quit for 1 week or if the user needs to authorize to install the update (due to lack of write permission).

For backgrounded applications (apps that do not appear in the Dock), Sparkle 2.2 onwards will not let a scheduled update alert steal focus from another application that may be active (with the exception of your application having just been launched). Update alerts will show up in the background behind other windows, even when the app is currently active.

If you want your application to deliver scheduled alerts in a subtle manner, you may opt into Sparkle's Gentle Reminders APIs. These are a part of [SPUStandardUserDriverDelegate](/documentation/api-reference/Protocols/SPUStandardUserDriverDelegate.html), which is a part of Sparkle's standard user interface.

## Gentle Reminders APIs

These APIs can be used to implement gentle reminders for your app:

* `-[SPUStandardUserDriverDelegate supportsGentleScheduledUpdateReminders]` declares support for implementing gentle reminders
* `-[SPUStandardUserDriverDelegate standardUserDriverShouldHandleShowingScheduledUpdate:andInImmediateFocus:]` is the method to implement to override Sparkle's handling of showing a new scheduled update with your own handling if desired
* `-[SPUStandardUserDriverDelegate standardUserDriverWillHandleShowingUpdate:forUpdate:state:]` is the method to implement to add additional update reminders before the update will be shown by the standard user driver or by you.
* `-[SPUStandardUserDriverDelegate standardUserDriverDidReceiveUserAttentionForUpdate:]` lets your app know when the user has given attention to a new update alert
* `-[SPUStandardUserDriverDelegate standardUserDriverWillFinishUpdateSession]` lets your app know when the user session for handling a new update will finish

## Gentle Reminders Examples

These examples show how to implement gentle reminders using Sparkle 2.2 or later.

They also include a bit of test / debugging code for testing gentle reminders by performing a scheduled update check in the background 10 seconds after the app launched. Within this time period, you can test the gentle update reminders when the app is active or inactive. Do not use test code like this in production. Let Sparkle handle scheduling update checks automatically instead.

### Window Title Accessory Example

The first example demonstrates creating a gentle reminder when a new update alert is available by attaching an "Update Available" button to the application's main window's titlebar.

This example lets Sparkle handle showing update alerts when Sparkle wants to show the update in immediate and utmost focus. Otherwise, this example overrides showing new update alerts and creates an additional UI reminder.

The application can override Sparkle's default behavior in cases where the application believes it can provide reminders in a more gentle manner. 

```swift
@NSApplicationMain
@objc class AppDelegate: NSObject, NSApplicationDelegate, SPUStandardUserDriverDelegate {
    // This controller's updaterDelegate and userDriverDelegate outlets are connected to this instance as well
    @IBOutlet var updaterController: SPUStandardUpdaterController!
    // Using a window outlet for demonstrating purposes. A real app may manage a window controller.
    @IBOutlet var window: NSWindow!
    // A view controller we attach to our window's titlebar when updates are available
    var titlebarAccessoryViewController: NSTitlebarAccessoryViewController? = nil
    
    func applicationDidFinishLaunching(_ notification: Notification) {
        #if DEBUG // "-D DEBUG" is added for Debug in Other Swift Flags
            // This snippet is only for testing gentle scheduled reminders
            // Remove this in production to respect Sparkle's update scheduler
            // You may also want to test with no delay for testing near app launch experience
            DispatchQueue.main.asyncAfter(deadline: .now() + 10.0) {
                self.updaterController.updater.checkForUpdatesInBackground()
            }
        #endif
    }
    
    // Declares that we support gentle scheduled update reminders to Sparkle's standard user driver
    var supportsGentleScheduledUpdateReminders: Bool {
        return true
    }
    
    func standardUserDriverShouldHandleShowingScheduledUpdate(_ update: SUAppcastItem, andInImmediateFocus immediateFocus: Bool) -> Bool {
        // If the standard user driver will show the update in immediate focus (e.g. near app launch),
        // then let Sparkle take care of showing the update.
        // Otherwise we will handle showing any other scheduled updates
        return immediateFocus
    }
    
    func standardUserDriverWillHandleShowingUpdate(_ handleShowingUpdate: Bool, forUpdate update: SUAppcastItem, state: SPUUserUpdateState) {
        // We will ignore updates that the user driver will handle showing
        // This includes user initiated (non-scheduled) updates
        guard !handleShowingUpdate else {
            return
        }
        
        // Attach a gentle UI indicator on our window
        do {
            let updateButton = NSButton(frame: NSMakeRect(0, 0, 120, 100))
            updateButton.title = "Update Available"
            updateButton.bezelStyle = .recessed
            updateButton.target = updaterController
            updateButton.action = #selector(updaterController.checkForUpdates(_:))
            
            let accessoryViewController = NSTitlebarAccessoryViewController()
            accessoryViewController.layoutAttribute = .right
            accessoryViewController.view = updateButton
            
            self.window.addTitlebarAccessoryViewController(accessoryViewController)
            
            titlebarAccessoryViewController = accessoryViewController
        }
    }
    
    func standardUserDriverWillFinishUpdateSession() {
        // We will dismiss our gentle UI indicator if the user session for the update finishes
        titlebarAccessoryViewController?.removeFromParent()
        titlebarAccessoryViewController = nil
    }
}
```

This example can be altered to suit an application's policy. For example, a particular application may for want to also override handling showing scheduled update alerts even when Sparkle wants to show them in immediate focus, or still add an additional UI indicator when the user initiated an update check.

### Background App and Dock Badge Example

This second example demonstrates a background running, or dockless, app creating a gentle update reminder by:
* Bringing the app back in the Dock as a regular foreground application
* Adding a badge alert label to the application's Dock icon
* Posting a user notification to Notification Center

The approach in this example is to bring the app into the Dock, along with a badge indicator, which will gently let the user know an update is available. The application can go back in the background once the user is finished with installing or dismissing the update. During the update session, this will allow the application's update and its windows to be available in the application switcher (via command tab), and make bringing back the update easy.

This example also posts a user notification that a new update is available. Note posting a notification to Notification Center is an auxiliary but not definitive way of notifying users of updates. There is no guarantee the notification will be delivered and that the user will see the notification. It is also common for a user to not approve your app from delivering notifications.

Prior versons of Sparkle could steal focus from other running applications when new scheduled updates became available. This example instead provides gentle reminders using the Dock and Notification Center without overriding when Sparkle shows the update alert.

```swift
let UPDATE_NOTIFICATION_IDENTIFIER = "UpdateCheck"

@NSApplicationMain
@objc class AppDelegate: NSObject, NSApplicationDelegate, SPUStandardUserDriverDelegate, UNUserNotificationCenterDelegate {
    // This controller's updaterDelegate and userDriverDelegate outlets are connected to this instance as well
    @IBOutlet var updaterController: SPUStandardUpdaterController!
    // Menu bar item for our background application
    var statusBarItem: NSStatusItem?
    
    func applicationDidFinishLaunching(_ notification: Notification) {
        // Add a status bar item in the menu bar for our app
        do {
            let statusBarItem = NSStatusBar.system.statusItem(withLength: NSStatusItem.squareLength)
            statusBarItem.button?.title = "🌯"
            
            let statusBarMenu = NSMenu()
            statusBarItem.menu = statusBarMenu
            
            let menuItem = NSMenuItem()
            menuItem.title = "Check for Updates…"
            menuItem.target = updaterController
            menuItem.action = #selector(SPUStandardUpdaterController.checkForUpdates(_:))
            
            statusBarMenu.addItem(menuItem)
            
            self.statusBarItem = statusBarItem
        }
        
        // Make the app run in the background
        NSApp.setActivationPolicy(.accessory)
        
        #if DEBUG // "-D DEBUG" is added for Debug in Other Swift Flags
            // This snippet is only for testing gentle scheduled reminders
            // Remove this in production to respect Sparkle's update scheduler
            // You may also want to test with no delay for testing near app launch experience
            DispatchQueue.main.asyncAfter(deadline: .now() + 10.0) {
                self.updaterController.updater.checkForUpdatesInBackground()
            }
        #endif
        
        UNUserNotificationCenter.current().delegate = self
    }
    
    // Request for permissions to publish notifications for update alerts
    // This delegate method will be called when Sparkle schedules an update check in the future,
    // which may be a good time to request for update permission. This will be after the user has allowed
    // Sparkle to check for updates automatically. If you need to publish notifications for other reasons,
    // then you may have a more ideal time to request for notification authorization unrelated to update checking.
    func updater(_ updater: SPUUpdater, willScheduleUpdateCheckAfterDelay delay: TimeInterval) {
        UNUserNotificationCenter.current().requestAuthorization(options: [.badge, .alert, .sound]) { granted, error in
            // Examine granted outcome and error if desired...
        }
    }
    
    // Declares that we support gentle scheduled update reminders to Sparkle's standard user driver
    var supportsGentleScheduledUpdateReminders: Bool {
        return true
    }
    
    func standardUserDriverWillHandleShowingUpdate(_ handleShowingUpdate: Bool, forUpdate update: SUAppcastItem, state: SPUUserUpdateState) {
        // When an update alert will be presented, place the app in the foreground
        // We will do this for updates the user initiated themselves too for consistency
        // When we later post a notification, the act of clicking a notification will also change the app
        // to have a regular activation policy. For consistency, we should do this if the user
        // does not click on the notification too.
        NSApp.setActivationPolicy(.regular)
        
        if !state.userInitiated {
            // And add a badge to the app's dock icon indicating one alert occurred
            NSApp.dockTile.badgeLabel = "1"
            
            // Post a user notification
            // For banner style notification alerts, this may only trigger when the app is currently inactive.
            // For alert style notification alerts, this will trigger when the app is active or inactive.
            do {
                let versionString = update.displayVersionString ?? update.versionString

                let content = UNMutableNotificationContent()
                content.title = "A new update is available"
                content.body = "Version \(versionString) is now available"
                
                let request = UNNotificationRequest(identifier: UPDATE_NOTIFICATION_IDENTIFIER, content: content, trigger: nil)
                
                UNUserNotificationCenter.current().add(request)
            }
        }
    }
    
    func standardUserDriverDidReceiveUserAttention(forUpdate update: SUAppcastItem) {
        // Clear the dock badge indicator for the update
        NSApp.dockTile.badgeLabel = ""
        
        // Dismiss active update notifications if the user has given attention to the new update
        UNUserNotificationCenter.current().removeDeliveredNotifications(withIdentifiers: [UPDATE_NOTIFICATION_IDENTIFIER])
    }
    
    func standardUserDriverWillFinishUpdateSession() {
        // Put app back in background when the user session for the update finished.
        // We don't have a convenient reason for the user to easily activate the app now.
        // Note this assumes there's no other windows for the app to show
        NSApp.setActivationPolicy(.accessory)
    }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        if response.actionIdentifier == UNNotificationDefaultActionIdentifier {
            // If the notificaton is clicked on, make sure we bring the update in focus
            // If the app is terminated while the notification is clicked on,
            // this will launch the application and perform a new update check.
            // This can be more likely to occur if the notification alert style is Alert rather than Banner
            updaterController.checkForUpdates(nil)
        }
        
        completionHandler()
    }
}
```

## Alternative User Interfaces

The APIs for gentle reminders discussed here are a part of Sparkle's standard user interface and are great if you want to leverage Sparkle's standard UI. If you need even greater customization, you can implement your own custom user interface by writing your your own [SPUUserDriver](/documentation/api-reference/Protocols/SPUUserDriver.html) and passing it when you instantiate a [`SPUUpdater`](/documentation/api-reference/Classes/SPUUpdater.html). 
