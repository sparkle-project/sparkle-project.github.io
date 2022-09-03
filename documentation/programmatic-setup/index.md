---
layout: documentation
id: documentation
title: Setting up Sparkle programmatically
---

## Create an Updater in Cocoa programmatically

This is an example of creating an updater in Sparkle 2 with Cocoa programmatically. It hooks up a menu item's target/action for checking for updates.

```swift
import Cocoa
import Sparkle

@NSApplicationMain
@objc class AppDelegate: NSObject, NSApplicationDelegate {
    // Hook up this menu item outlet in Interface Builder. The menu item's title is typically "Check for Updates…"
    @IBOutlet var checkForUpdatesMenuItem: NSMenuItem!
    
    let updaterController: SPUStandardUpdaterController
    
    override init() {
        // If you want to start the updater manually, pass false to startingUpdater and call .startUpdater() later
        // This is where you can also pass an updater delegate if you need one
        updaterController = SPUStandardUpdaterController(startingUpdater: true, updaterDelegate: nil, userDriverDelegate: nil)
    }
    
    func applicationDidFinishLaunching(_ notification: Notification) {
        // Hooking up the menu item's target/action to the standard updater controller does two things:
        // 1. The menu item's action is set to perform a user-initiated check for new updates
        // 2. The menu item is enabled and disabled by the updater controller depending on -[SPUUpdater canCheckForUpdates]
        checkForUpdatesMenuItem.target = updaterController
        checkForUpdatesMenuItem.action = #selector(SPUStandardUpdaterController.checkForUpdates(_:))
    }
}
```

## Create an Updater in SwiftUI

This is an example of creating an updater in Sparkle 2 with SwiftUI. It creates a new menu item allowing users to check for new updates and ensures its disabled state is updated.

```swift
import SwiftUI
import Sparkle

// This view model class manages Sparkle's updater and publishes when new updates are allowed to be checked
final class UpdaterViewModel: ObservableObject {
    private let updaterController: SPUStandardUpdaterController
    
    @Published var canCheckForUpdates = false
    
    init() {
        // If you want to start the updater manually, pass false to startingUpdater and call .startUpdater() later
        // This is where you can also pass an updater delegate if you need one
        updaterController = SPUStandardUpdaterController(startingUpdater: true, updaterDelegate: nil, userDriverDelegate: nil)
        
        updaterController.updater.publisher(for: \.canCheckForUpdates)
            .assign(to: &$canCheckForUpdates)
    }
    
    func checkForUpdates() {
        updaterController.checkForUpdates(nil)
    }
}

// This additional view is needed for the disabled state on the menu item to work properly before Monterey.
// See https://stackoverflow.com/questions/68553092/menu-not-updating-swiftui-bug for more information
struct CheckForUpdatesView: View {
    @ObservedObject var updaterViewModel: UpdaterViewModel
    
    var body: some View {
        Button("Check for Updates…", action: updaterViewModel.checkForUpdates)
            .disabled(!updaterViewModel.canCheckForUpdates)
    }
}

@main
struct MyApp: App {
    @StateObject var updaterViewModel = UpdaterViewModel()
    
    var body: some Scene {
        WindowGroup {
        }
        .commands {
            CommandGroup(after: .appInfo) {
                CheckForUpdatesView(updaterViewModel: updaterViewModel)
            }
        }
    }
}
```

## Create an Updater in Mac Catalyst

This is an example for creating an updater in Sparkle 2 with Mac Catalyst. The code illustrated below is inside an AppKit bundle plug-in that is loaded by the Catalyst application using a shared `PlugIn` protocol. This snippet is missing plenty of details including adding a menu item and inter-opt between the host application and plug-in.

```swift
import AppKit
import Foundation
import Sparkle

@objc class AppKitPlugin: NSObject, PlugIn {
    let updaterController: SPUStandardUpdaterController
    
    required override init() {
        // We may want to defer starting the updater later, so we will pass false to startingUpdater
        // This is where you can also pass an updater delegate if you need one
        updaterController = SPUStandardUpdaterController(startingUpdater: false, updaterDelegate: nil, userDriverDelegate: nil)
    }
    
    // Called from the Catalyst app
    func startUpdater() {
        updaterController.startUpdater()
    }
}

```

Note Sparkle's standard user interface cannot display release notes because macOS WebKit views cannot be used inside a Catalyst application. A simple approach to work around this is to disable showing release notes via setting [SUShowReleaseNotes](/documentation/customization) to `NO`. A more advanced approach using Sparkle 2 is instantiating [SPUUpdater](/documentation/api-reference/Classes/SPUUpdater.html) with your own [SPUUserDriver](/documentation/api-reference/Protocols/SPUUserDriver.html) and custom user interface, which may use an iOS WebKit view from the Catalyst side.

If you run into issues with loading the Sparkle framework from your plug-in at runtime, please refer to [Add the Sparkle framework to your project](/documentation/#1-add-the-sparkle-framework-to-your-project) where the setup guide talks about having the <samp>Runpath Search Paths</samp> set to `@loader_path/../Frameworks` and using an `Apple Development` certificate for development to satisfy Library Validation. This applies to the setup for the plug-in target as well.

## Create an Updater in Qt

This is an example of creating an updater in Sparkle 2 with Qt and uses [Key-Value Observing (KVO)](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html) to ensure the check for update's menu item state is updated. This example can also be used as reference when integrating Sparkle into other non-Apple provided toolkits.

```objectivec++
// File: updater.h
// Description: Header interface for your application's updater. Implementation could differ by platform.

#ifndef UPDATER_H
#define UPDATER_H

#include <QObject>

class QAction;

#ifdef __OBJC__
@class AppUpdaterDelegate;
#endif

class Updater : public QObject
{
public:
    // Caller example from a QMainWindow subclass:
    /*
     QAction *updaterAction = new QAction("&Check for Updates…", this);
     updaterAction->setMenuRole(QAction::ApplicationSpecificRole);
     
     QMenu *updaterMenu = menuBar()->addMenu("&Updater");
     updaterMenu->addAction(updaterAction);
    
     Updater *updater = new Updater(updaterAction);
     */
    // This constructor initializes the updater,
    // and takes care of connecting the check for updates action and updating its enabled state
    // Note: this must be called on the main thread
    Updater(QAction *checkForUpdatesAction);

private slots:
    void checkForUpdates();

private:
#ifdef __OBJC__
    // Expose ObjC type only to updater_sparkle.mm. This allows ARC to properly track its lifetime.
    AppUpdaterDelegate *_updaterDelegate;
#else
    void *_updaterDelegate;
#endif
};

#endif
```

```objectivec++
// File: updater_sparkle.mm
// Description: macOS and Sparkle implementation for the application's updater
// Notes: Compile updater_sparkle.mm with -fobjc-arc because this file assumes Automatic Reference Counting (ARC) is enabled

#include "updater.h"
#include <qaction.h>

#import <Sparkle/Sparkle.h>

// An updater delegate class that mainly takes care of updating the check for updates menu item's state
// This class can also be used to implement other updater delegate methods
@interface AppUpdaterDelegate : NSObject <SPUUpdaterDelegate>

@property (nonatomic) SPUStandardUpdaterController *updaterController;

@end

@implementation AppUpdaterDelegate

- (void)observeCanCheckForUpdatesWithAction:(QAction *)action
{
    [_updaterController.updater addObserver:self forKeyPath:NSStringFromSelector(@selector(canCheckForUpdates)) options:(NSKeyValueObservingOptionInitial | NSKeyValueObservingOptionNew) context:(void *)action];
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey, id> *)change context:(void *)context
{
    if ([keyPath isEqualToString:NSStringFromSelector(@selector(canCheckForUpdates))]) {
        QAction *menuAction = (QAction *)context;
        menuAction->setEnabled(_updaterController.updater.canCheckForUpdates);
    } else {
        [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
    }
}

- (void)dealloc
{
    @autoreleasepool {
        [_updaterController.updater removeObserver:self forKeyPath:NSStringFromSelector(@selector(canCheckForUpdates))];
    }
}

@end

// Creates and starts the updater. There's nothing else required.
Updater::Updater(QAction *checkForUpdatesAction)
{
    @autoreleasepool {
        _updaterDelegate = [[AppUpdaterDelegate alloc] init];
        _updaterDelegate.updaterController = [[SPUStandardUpdaterController alloc] initWithStartingUpdater:YES updaterDelegate:_updaterDelegate userDriverDelegate:nil];
        
        connect(checkForUpdatesAction, &QAction::triggered, this, &Updater::checkForUpdates);
        
        [_updaterDelegate observeCanCheckForUpdatesWithAction:checkForUpdatesAction];
    }
}

// Called when the user checks for updates
void Updater::checkForUpdates()
{
    @autoreleasepool {
        [_updaterDelegate.updaterController checkForUpdates:nil];
    }
}
```

Specific steps on setting up an application and incorporating a framework using an external build system are outside the scope of this document. However, on the lower level you will need to at least:

* Add `updater_sparkle.mm` to source files to compile on macOS using `-fobjc-arc` compile flags and add `updater.h` to your project too
* Link against your copy of Sparkle (`-framework Sparkle`)
* Set the framework search path to the parent directory that contains `Sparkle.framework` (`-F/path/to/dir/`)
* Add a runtime search path (rpath) so your application searches for Sparkle in your app's Frameworks directory (`-Wl,-rpath,@loader_path`). Use `otool -l <executable-path>` and `otool -L <executable-path>` to ensure all paths your application references are self-contained or provided from the macOS system.
* Copy `Sparkle.framework` into your built application bundle in `Contents/Frameworks/` and make sure symlinks are properly preserved
* Set the `CFBundleIdentifier`, `CFBundleVersion`, and `CFBundleShortVersionString` keys in your application's Info.plist
* Set the `SUFeedURL` and `SUPublicEDKey` keys in your app's `Info.plist` described in [later documentation sections](/documentation/#3-segue-for-security-concerns)
* Set the deployment target to your application (`-mmacosx-version-min=version` flag and `LSMinimumSystemVersion` Info.plist key) and build architectures (e.g. `-arch arm64 -arch x86_64`)

Build systems may provide higher-level constructs to set these details. For example, `cmake`'s [`find_library`](https://cmake.org/cmake/help/latest/command/find_library.html) command on a framework will automatically add `-framework A` and `-F<fullPath>` and `cmake`'s [`MACOSX_BUNDLE_INFO_PLIST`](https://cmake.org/cmake/help/latest/prop_tgt/MACOSX_BUNDLE_INFO_PLIST.html) property can be used to provide custom Info.plist keys.

## Core APIs

In Sparkle 2 you can also choose to instantiate and use [SPUUpdater](/documentation/api-reference/Classes/SPUUpdater.html) directly instead of the [SPUStandardUpdaterController](/documentation/api-reference/Classes/SPUStandardUpdaterController.html) wrapper if you need more control over your user interface or what bundle to update.

If you use another UI toolkit, these are the relevant APIs in Sparkle 2 for checking for updates and handling menu item validation:

* [-[SPUStandardUpdaterController startUpdater]](/documentation/api-reference/Classes/SPUStandardUpdaterController.html#/c:objc(cs)SPUStandardUpdaterController(im)startUpdater) or [-[SPUUpdater startUpdater:]](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(im)startUpdater:) for starting the updater (unless it is specified to be automatically started)
* [-[SPUStandardUpdaterController checkForUpdates:]](/documentation/api-reference/Classes/SPUStandardUpdaterController.html#/c:objc(cs)SPUStandardUpdaterController(im)checkForUpdates:) or [-[SPUUpdater checkForUpdates]](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(im)checkForUpdates) for the user to check for updates
* [-[SPUUpdater canCheckForUpdates]](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(py)canCheckForUpdates) for menu item validation and knowing when the user can check for updates. This property is also [KVO compliant and its value changes are observable](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueObserving/Articles/KVOBasics.html#//apple_ref/doc/uid/20002252-BAJEAIEE).

If you are using Sparkle 1, you will need to use these APIs:

* `+[SUUpdater sharedUpdater]` for creating and starting the updater automatically
* `-[SUUpdater checkForUpdates:]` for the user to check for updates
* `-[SUUpdater validateMenuItem:]` for menu item validation

## API Expectations

Prefer to set the updater's initial properties in your bundle's Info.plist as described in [Customizing Sparkle](/documentation/customization/). This includes properties like the updater's feed URL and its update checking behavior. Only configuring the feed URL and signing keys (via `SUFeedURL` and `SUPublicEDKey`) in your bundle's Info.plist is strongly recommended, which are described in [later documentation sections](/documentation/#3-segue-for-security-concerns).

Only set [SPUUpdater](/documentation/api-reference/Classes/SPUUpdater.html) (or `SUUpdater` for Sparkle 1) properties, related to update checking, programatically when the user wants to make updater setting changes, otherwise you may be ignoring the user's preferences and resetting the updater's cycle unnecessarily. Similarly, developers shouldn't maintain their own set of user defaults on top of [SPUUpdater](/documentation/api-reference/Classes/SPUUpdater.html) (or `SUUpdater`) properties that are documented to already be backed by `NSUserDefaults` from user setting changes.

If you want to switch to an alternative feed at runtime, please check our section on [setting the feed programmatically](/documentation/publishing#setting-the-feed-programmatically) first on the recommended API usage (which is not through `-setFeedURL:`).

Avoid forcing a manual [-checkForUpdatesInBackground](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(im)checkForUpdatesInBackground) unless your application requires this check at specific points of time. If used incorrectly, this can interfere with Sparkle's default behavior for asking the user's permission to check for updates automatically, and for calling this method periodically without probing too often. If you do require a manual background update check, either ensure [-automaticallyChecksForUpdates](api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(py)automaticallyChecksForUpdates) is already `YES` (but don't set the property yourself in this context) or set `SUEnableAutomaticChecks` Info.plist key described in [Customizing Sparkle](/documentation/customization/) to explicitly opt out of asking the user permission to check for updates automatically. Finally, note that by default a new update that is found from a background check is not guaranteed to be shown to the user immediately (due to poor connectivity, downloading/installing an update silently in the background, or waiting until a more opportune time to alert the user).
