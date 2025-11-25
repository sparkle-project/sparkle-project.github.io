---
layout: documentation
id: documentation
title: Adding a Settings UI
---

## Adding a Settings UI in Cocoa

These instructions below are for adding settings in a Cocoa nib and assume you have a Settings nib with its `File's Owner` being set to your Settings controller class (e.g, a `NSWindowController` subclass). The controller class should have an `updater` property set to your application's `SPUUpdater`, which may be passed from your application's delegate.

### Enable automatic checking

* Drag a check button from the Library to your document.
* Set the title to something like "Automatically check for updates".
* Select the check button, and under the Bindings tab, select Value.
* Check the Bind to: check button, choose `File's Owner` from the popup, and set the Model Key Path to `updater.automaticallyChecksForUpdates`.

### Update check interval

* Drag a popup button from the Library to your document.
* Set the titles of the menu items to e.g. "Hourly", "Daily", "Weekly", "Monthly".
* Set the tags of the menu items to the corresponding times in seconds, e.g. 3600, 86400, 604800, 2629800.
* Select the popup button, and under the Bindings tab, select Selected Tag.
* Check the Bind to: check button, choose `File's Owner` from the popup, and set the Model Key Path to `updater.updateCheckInterval`.
* Select Enabled, check the Bind to: check button, choose `File's Owner` from the popup, and set the Model Key Path to `updater.automaticallyChecksForUpdates`.

### Other settings

Follow directions similar to [Enable automatic checking](#enable-automatic-checking) to bind a check button to `updater.automaticallyDownloadsUpdates` or `updater.sendsSystemProfile`. See [customization](/documentation/customization/#infoplist-settings) for details on the available keys.

## Adding Settings in SwiftUI

This is a continuation from [Creating an Updater in SwiftUI](/documentation/programmatic-setup#create-an-updater-in-swiftui).

Create a new view for the updater settings:

```swift
// This is the view for our updater settings
// It manages local state for checking for updates and automatically downloading updates
// Upon user changes to these, the updater's properties are set. These are backed by NSUserDefaults.
// Note the updater properties should *only* be set when the user changes the state.
struct UpdaterSettingsView: View {
    private let updater: SPUUpdater
    
    @State private var automaticallyChecksForUpdates: Bool
    @State private var automaticallyDownloadsUpdates: Bool
    
    init(updater: SPUUpdater) {
        self.updater = updater
        self.automaticallyChecksForUpdates = updater.automaticallyChecksForUpdates
        self.automaticallyDownloadsUpdates = updater.automaticallyDownloadsUpdates
    }
    
    var body: some View {
        VStack {
            Toggle("Automatically check for updates", isOn: $automaticallyChecksForUpdates)
                .onChange(of: automaticallyChecksForUpdates) { newValue in
                    updater.automaticallyChecksForUpdates = newValue
                }
            
            Toggle("Automatically download updates", isOn: $automaticallyDownloadsUpdates)
                .disabled(!automaticallyChecksForUpdates)
                .onChange(of: automaticallyDownloadsUpdates) { newValue in
                    updater.automaticallyDownloadsUpdates = newValue
                }
        }.padding()
    }
}
```

Then add these settings to your `App`'s `body`:

```swift
var body: some Scene {
    /* ... */
    Settings {
        UpdaterSettingsView(updater: updaterController.updater)
    }
}
```

## Adding Settings in Qt

This is a continuation from [Creating an Updater in Qt](/documentation/programmatic-setup#create-an-updater-in-qt) and illustrates one way of adding updater settings.

Add new methods for connecting and setting the updater's [automaticallyChecksForUpdates](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(py)automaticallyChecksForUpdates) property from a `QCheckBox`:


```objectivec++
// File: updater.h

/* ... */

class QAction;
class QCheckBox;

class Updater : public QObject
{
    Q_OBJECT

public:
    Updater(QAction *checkForUpdatesAction);
    void connectAutomaticallyCheckForUpdatesButton(QCheckBox *checkBox);
private slots:
    void checkForUpdates();
    void setAutomaticallyCheckForUpdates(int newState);

    /* ... */
}

```

Then implement the new methods:

```objectivec++
#include "updater.h"
#include <qaction.h>
#include <qcheckbox.h>

#import <Sparkle/Sparkle.h>

/* ... */

void Updater::connectAutomaticallyCheckForUpdatesButton(QCheckBox *checkBox)
{
    @autoreleasepool {
        checkBox->setChecked(_updaterDelegate.updaterController.updater.automaticallyChecksForUpdates ? Qt::CheckState::Checked : Qt::CheckState::Unchecked);
        connect(checkBox, &QCheckBox::stateChanged, this, &Updater::setAutomaticallyCheckForUpdates);
    }
}

// Called when the user toggles automatically check for updates checkbox
void Updater::setAutomaticallyCheckForUpdates(int newState)
{
    @autoreleasepool {
        _updaterDelegate.updaterController.updater.automaticallyChecksForUpdates = (newState == Qt::CheckState::Checked);
    }
}
```

Note that [automaticallyChecksForUpdates](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(py)automaticallyChecksForUpdates) is an updater setting that is backed by `NSUserDefaults`, and shouldn't be backed by additional defaults. Also note this property should only be set upon the user toggling the checkbox.

---

## Watch out for Preference Caching

macOS caches plist files in `~/Library/Preferences`, so don't edit them directly. If you want to tweak these files for testing (e.g. change last update check date), use the `defaults` command.
