---
layout: documentation
id: documentation
title: Gentle Update Reminders
---

**Note**: This article is for Sparkle 2.2, which is currently unreleased.

### Scheduled Update Checks

By default, when your application launches for the first time Sparkle is not granted permission to make any update checks. On the second launch of your application, Sparkle prompts the user asking if it is allowed to check for new updates automatically. This is defined by the [`SUEnableAutomaticChecks`](/documentation/customization) setting.

Once Sparkle is granted permission, it is programmed to schedule update checks on a regular basis defined by the [`SUScheduledCheckInterval`](/documentation/customization/) setting.

Letting Sparkle handle checking for updates automatically provides two benefits:
* It ensures your server is not polled too often (for when an app is re-launched frequently)
* It ensures update checks trigger often enough (for when your app has not quit for a long time)

Keeping your users up to date is important, however users do not want to have their focus stolen from new update alerts at inopportune times.

As of Sparkle 2.2, Sparkle prioritizes showing scheduled update alerts at opportune times when:
* The user just launched your app or just granted your app permission to check for updates automatically
* The user's system has been idle for some time
* The user comes back to your application from another application (preferred over showing an alert when a user is actively using your app)

Note even for updates downloaded automatically and silently in the background, Sparkle may show an update alert to the user if the application hasn't quit for quite a long time or if the user needs to authorize to install the update (due to lack of write permission).

For backgrounded applications (apps that do not appear in the Dock), Sparkle 2.2 onwards will not let a scheduled update alert steal focus from another application that may be active (with the exception of your application having just been launched). As a consequence, users may have a difficult time noticing update alerts scheduled to occur some time after your application has been launched. Some backgrounded applications using prior versions of Sparkle may have disabled automatic update checks and only scheduled checking for new updates on launch. But now, Sparkle 2.2 provides an alternative solution.

If you want your application to deliver scheduled alerts in a more subtle manner or if you want your scheduled alerts to stand out more (e.g. for backgrounded apps), you may opt into Sparkle's Gentle Reminders APIs. These are a part of [SPUStandardUserDriverDelegate](/documentation/api-reference/Protocols/SPUStandardUserDriverDelegate.html), which is a part of Sparkle's standard user interface.

### Gentle Reminders APIs

These APIs can be used to implement gentle reminders for your app:

* `-[SPUStandardUserDriverDelegate supportsGentleScheduledUpdateReminders]` declares support for implementing gentle reminders
* `-[SPUStandardUserDriverDelegate standardUserDriverShouldShowUpdateAlertForScheduledUpdate:inFocusNow:]` is the method to implement to add additional reminders and override Sparkle's handling of showing a new scheduled update alert if desired
* `-[SPUStandardUserDriverDelegate standardUserDriverWillCloseUpdateAlertForUpdate:]` lets your app know when the user will close the update alert
* `-[SPUStandardUserDriverDelegate standardUserDriverDidMakeUpdateAlertWindowKeyForUpdate:]` lets your app know when the update alert window has become the key or active window

### Gentle Reminders Example

This is an example of implementing gentle reminders using Sparkle 2.2 ownards. It demonstrates creating two UI gentle reminders when a new update alert is available by:
* Attaching an "Update Available" button to the application's main window's titlebar (Primary)
* Posting a notification to Notification Center when the app is in the background (Secondary)

Note that posting a notification to Notification Center is an auxiliary but not definitive way of notifying users of updates. There is no guarantee the notification will be delivered and that the user will see the notification. For example, it is common for a user to not approve your app from delivering notifications. This is why user notifications are considered a more "secondary" reminder.

This example lets Sparkle handle showing update alerts when Sparkle thinks it's a good time to show the update in utmost focus. Otherwise, this example overrides showing new update alerts and creates additional UI reminders. This is handled in `-[SPUStandardUserDriverDelegate standardUserDriverShouldShowUpdateAlertForScheduledUpdate:inFocusNow:]`.

This example also includes a bit of test / debugging code for testing gentle reminders by performing a scheduled update check in the background 10 seconds after the app launched. Within this time period, you can test the gentle update reminders when the app is active or inactive. Do not use test code like this in production. Let Sparkle handle scheduling update checks automatically instead.

```swift
let UPDATE_NOTIFICATION_IDENTIFIER = "UpdateCheck"

@NSApplicationMain
@objc class AppDelegate: NSObject, NSApplicationDelegate, SPUUpdaterDelegate, SPUStandardUserDriverDelegate, UNUserNotificationCenterDelegate {
    // This controller's updaterDelegate and userDriverDelegate outlets are connected to this instance as well
    @IBOutlet var updaterController: SPUStandardUpdaterController!
    @IBOutlet var window: NSWindow!
    
    var titlebarAccessoryViewController: NSTitlebarAccessoryViewController? = nil
    
    func applicationDidFinishLaunching(_ notification: Notification) {
        #if DEBUG // "-D DEBUG" is added for Debug in Other Swift Flags
            // This snippet is only for testing gentle scheduled reminders
            // Remove this in production to respect Sparkle's update scheduler
            DispatchQueue.main.asyncAfter(deadline: .now() + 10.0) {
                self.updaterController.updater.checkForUpdatesInBackground()
            }
        #endif
        
        // Should be set very early on in application's lifecycle, otherwise notifications could be missed. This is a good place.
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
    
    func standardUserDriverShouldShowUpdateAlert(forScheduledUpdate update: SUAppcastItem, inFocusNow: Bool) -> Bool {
        // If the update will be shown in active focus (e.g. near app launch), let Sparkle take care of handling the update alert.
        // We won't provide additional reminders in this case
        // Although, you could choose to act differently if you desire
        if inFocusNow {
            return true
        }
        
        // We will take over showing a gentle update reminder
        
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
        
        // Post a user notification which will only trigger when app is not active
        do {
            let versionString = update.displayVersionString ?? update.versionString

            let content = UNMutableNotificationContent()
            content.title = "A new update is available"
            content.body = "Version \(versionString) is now available"
            
            let request = UNNotificationRequest(identifier: UPDATE_NOTIFICATION_IDENTIFIER, content: content, trigger: nil)
            
            UNUserNotificationCenter.current().add(request)
        }
        
        return false
    }
    
    private func removeDeliveredUpdateNotifications() {
        UNUserNotificationCenter.current().removeDeliveredNotifications(withIdentifiers: [UPDATE_NOTIFICATION_IDENTIFIER])
    }
    
    func standardUserDriverWillCloseUpdateAlert(forUpdate update: SUAppcastItem) {
        // We will choose to dismiss our gentle UI indicator if the user closes the update alert
        titlebarAccessoryViewController?.removeFromParent()
        titlebarAccessoryViewController = nil
        
        // Dismiss active notifications too
        removeDeliveredUpdateNotifications()
    }
    
    func standardUserDriverDidMakeUpdateAlertWindowKey(forUpdate update: SUAppcastItem) {
        // Dismiss active notifications once user has the update alert window as the key window
        removeDeliveredUpdateNotifications()
    }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        if response.actionIdentifier == UNNotificationDefaultActionIdentifier {
            // If the notificaton is clicked on, make sure we bring the update in focus
            // If the app is terminated while the notification is delivered and clicked by the user,
            // this will then launch the application and perform a new update check
            updaterController.checkForUpdates(nil)
        }
        
        completionHandler()
    }
}
```

### Alternative User Interfaces

The APIs for gentle reminders discussed here are a part of Sparkle's standard user interface and great if you want to leverage Sparkle's UI. If you need even greater customization, you can implement your own custom user interface by writing your your own [SPUUserDriver](/documentation/api-reference/Protocols/SPUUserDriver.html) and passing it when you instantiate a [`SPUUpdater`](/documentation/api-reference/Classes/SPUUpdater.html). 
