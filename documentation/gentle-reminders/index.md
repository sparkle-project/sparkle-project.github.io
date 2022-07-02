---
layout: documentation
id: documentation
title: Gentle Update Reminders
---

## Gentle Update Reminders

While Sparkle's standard UI tries to provide update reminders to the user at opportune times, the gentle update reminder APIs in Sparkle 2.2 and later allow developers to provide and customize update reminders in a more flexible way.

### Scheduled Update Checks

Once Sparkle has permission to check for updates automatically, it is programmed to schedule update checks on a regular basis defined by the [`SUScheduledCheckInterval`](/documentation/customization/) setting. Sparkle handles this automatically without polling your server too often and by ensuring that users receive updates in a timely manner.

As of Sparkle 2.2, Sparkle prioritizes showing scheduled update alerts for regular (non-backgrounded) applications at opportune times when:
* The user just launched your app or interacted with the updater recently (for example, granting Sparkle permission to allow automatic update checks)
* The user's system has been idle for some time and no power assertion has been made by your app to prevent display sleep while your app is active
* The user comes back to your application from another application (instead of showing an alert when a user is actively using your app)

For backgrounded applications (apps that do not appear in the Dock), Sparkle 2.2 onwards will not let a scheduled update alert steal focus from another application or window that may be currently active -- with the one exception of when your application just launched. Scheduled update alerts that show up after launch will be presented behind other apps and windows.

Note even for updates downloaded silently in the background and installed after the app terminates, Sparkle may show an update alert to the user if the application hasn't quit for 1 week or if the user needs to authorize to install an update due to lack of sufficient write permissions. Other cases where an update is shown to the user is when an update is critical or informational only. Sparkle must be able to present updates to the user even if updates are set to automatically download.

If you want your application to deliver scheduled alerts in a more gentle yet noticeable manner, you may opt into Sparkle's Gentle Reminders APIs. These are a part of [SPUStandardUserDriverDelegate](/documentation/api-reference/Protocols/SPUStandardUserDriverDelegate.html), which is a part of Sparkle's standard user interface.

### Gentle Reminders APIs

These APIs can be used to implement gentle reminders for your app:

* [`-[SPUStandardUserDriverDelegate supportsGentleScheduledUpdateReminders]`](/documentation/api-reference/Protocols/SPUStandardUserDriverDelegate.html#/c:objc(pl)SPUStandardUserDriverDelegate(py)supportsGentleScheduledUpdateReminders) declares support for implementing gentle reminders. Background (dockless) running apps may receive a log warning about scheduling update checks and not implementing gentle reminders. At minimum, this method needs to be implemented.
* [`-[SPUStandardUserDriverDelegate standardUserDriverShouldHandleShowingScheduledUpdate:andInImmediateFocus:]`](https://sparkle-project.github.io/documentation/api-reference/Protocols/SPUStandardUserDriverDelegate.html#/c:objc(pl)SPUStandardUserDriverDelegate(im)standardUserDriverShouldHandleShowingScheduledUpdate:andInImmediateFocus:) is the method to implement to override Sparkle's handling of showing a new scheduled update with your own handling if desired.
* [`-[SPUStandardUserDriverDelegate standardUserDriverWillHandleShowingUpdate:forUpdate:state:]`](https://sparkle-project.github.io/documentation/api-reference/Protocols/SPUStandardUserDriverDelegate.html#/c:objc(pl)SPUStandardUserDriverDelegate(im)standardUserDriverWillHandleShowingUpdate:forUpdate:state:) is the method to implement to add additional update reminders before the update will be shown by the standard user driver or by its delegate.
* [`-[SPUStandardUserDriverDelegate standardUserDriverDidReceiveUserAttentionForUpdate:]`](https://sparkle-project.github.io/documentation/api-reference/Protocols/SPUStandardUserDriverDelegate.html#/c:objc(pl)SPUStandardUserDriverDelegate(im)standardUserDriverDidReceiveUserAttentionForUpdate:) lets your app know when the user has given attention to a new update alert.
* [`-[SPUStandardUserDriverDelegate standardUserDriverWillFinishUpdateSession]`](/documentation/api-reference/Protocols/SPUStandardUserDriverDelegate.html#/c:objc(pl)SPUStandardUserDriverDelegate(im)standardUserDriverWillFinishUpdateSession) lets your app know when the user session for handling a new update will finish.

### Gentle Reminders Examples

These examples show how to implement gentle reminders using Sparkle 2.2 or later.

For testing when an update is ready to be checked right away (near app launch), reset the last update check time before launching your app:

```sh
defaults delete my-app-bundle-id SULastCheckTime
```

For testing when an update will be checked in the background around 30 seconds from now (not near app launch), set the last update check time right before launching your app:

```sh
defaults write my-app-bundle-id SULastCheckTime -date "$(date -v-1d -v+30S)"
```

Note this sets the last update check time to one day prior plus 30 additional seconds. This assumes your app uses the default scheduled check interval time of 1 day, otherwise you may need to adjust this computed date accordingly.

#### Window Title Accessory Example

The first example demonstrates creating a gentle reminder when a new update alert is available by attaching an "Update Available" button to the application's main window's titlebar.

This example lets Sparkle handle showing update alerts when Sparkle wants to show a new update in immediate and utmost focus. Otherwise, this example overrides showing new update alerts and creates a gentle UI reminder.

The application overrides Sparkle's default behavior in cases where it believes reminders can be provided in a more gentle manner.

```swift
@NSApplicationMain
@objc class AppDelegate: NSObject, NSApplicationDelegate, SPUStandardUserDriverDelegate {
    // This controller's updaterDelegate and userDriverDelegate outlets are connected to this instance as well
    @IBOutlet var updaterController: SPUStandardUpdaterController!
    // Using a window outlet for demonstrating purposes. A real app may manage a window controller.
    @IBOutlet var window: NSWindow!
    // A view controller we attach to our window's titlebar when updates are available
    var titlebarAccessoryViewController: NSTitlebarAccessoryViewController? = nil
    
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
            updateButton.title = "v\(update.displayVersionString) Available"
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

This example can be altered to suit an application's policy. For example, a particular application may for want to also override handling showing scheduled update alerts even when Sparkle wants to show them in immediate focus, or still add an additional UI indicator when the user initiated an update check (`state.userInitiated`).

#### Background App, Dock, and Notification Example

This second example demonstrates a background running, or dockless, app creating a gentle update reminder by:
* Bringing the app back in the Dock as a regular foreground application
* Adding a badge alert label to the application's Dock icon
* Posting a user notification to Notification Center

The approach in this example is to bring the app into the Dock, along with a badge indicator, which will gently let the user know an update is available. The application can go back in the background once the user is finished with installing or dismissing the update. During the update session, this will allow the application's update and its windows to be available in the application switcher (via command tab), and make bringing back the update easy.

This example also posts a user notification that a new update is available. Note posting a notification to Notification Center is an auxiliary but not definitive way of notifying users of updates. There is no guarantee the notification will be delivered and that the user will see the notification. It is also common for a user to not approve your app from delivering notifications.

Prior versons of Sparkle could steal focus from other running applications when new scheduled updates became available for a background running application. This example instead adds gentle reminders using the Dock and Notification Center without overriding when Sparkle shows the update alert.

Note that regular non-background applications (i.e, apps that show up in the Dock) should not post reminders and notifications like in this example. They should instead wait until the user re-activates the application and only post reminders contained inside the app. This example is most suited for background applications that are in use but may never be re-activated by the user.

```swift
let UPDATE_NOTIFICATION_IDENTIFIER = "UpdateCheck"

@NSApplicationMain
@objc class AppDelegate: NSObject, NSApplicationDelegate, SPUUpdaterDelegate, SPUStandardUserDriverDelegate, UNUserNotificationCenterDelegate {
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
                let content = UNMutableNotificationContent()
                content.title = "A new update is available"
                content.body = "Version \(update.displayVersionString) is now available"
                
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
        if response.notification.request.identifier == UPDATE_NOTIFICATION_IDENTIFIER && response.actionIdentifier == UNNotificationDefaultActionIdentifier {
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

## Custom User Interfaces

The APIs for gentle reminders discussed here are a part of Sparkle's standard user interface and are great if you want to leverage Sparkle's standard UI. If you need even greater customization, you can implement your own custom user interface by writing your your own [SPUUserDriver](/documentation/api-reference/Protocols/SPUUserDriver.html) and passing it when you instantiate a [`SPUUpdater`](/documentation/api-reference/Classes/SPUUpdater.html). 
