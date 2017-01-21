---
layout: documentation
id: documentation
title: Documentation
---
## Basic Setup

Follow these simple steps, and you'll have your app auto-updating ASAP. Note that Sparkle does [not yet support](//github.com/{{ site.github_username }}/Sparkle/issues/363) sandboxed applications.

If your app already has an older version of Sparkle, see [upgrading from previous versions](/documentation/upgrading/).

### 0. Distributing your App

Before even updating your application, you will first want to consider the method used for distribution. If you use a [package installer](/documentation/package-updates/), you may skip reading this section.

We recommend the following:

* Ship your application as a dmg on your product's website. Add an `/Applications` symlink in your dmg, or otherwise encourage the user to copy the app out of it. For updating your app through Sparkle, other supported archive formats like zip are still okay.
* Avoid placing your app inside another folder in your archive. If you ship an archive that is not a dmg on your product's website, which we do not recommend, the archive should then not contain multiple items. An exception to this guideline is if you use a signed dmg, but please be careful - the system may not warn you if the dmg is not signed properly!
* Sign the dmg using macOS 10.11.5 or later if you have a Developer ID Application certificate available from Apple. If you attempt signing with an older OS, this will not work correctly. Be sure to verify the dmg is signed after downloading it from your server. Signed dmg's are backwards compatible to older systems.

These guidelines follow Apple's best practices and works best with App Translocation which affects macOS 10.12 and later. When an application is running from a dmg or is translocated, Sparkle will not be able to update your application. Therefore, you should distribute your application in a way that will most likely be moved by the user.

### 1. Add the Sparkle framework to your project

If you are using [CocoaPods](//cocoapods.org), then follow [step 1 for CocoaPods](/documentation/cocoapods/) instead.

* First, we'll link the Sparkle framework to your target:
  * Drag Sparkle.framework into the Frameworks folder of your Xcode project.
  * Be sure to check the "copy items into the destination group's folder" box in the sheet that appears.
  * Make sure the box is checked for your app's target in the sheet's Add to targets list.
* Now we'll make sure the framework is copied into your app bundle:
  * Click on your project in the Project Navigator.
  * Click your target in the project editor.
  * Click on the <samp>Build Phases</samp> tab.
  * Choose <samp>Editor › Add Build Phase › Add Copy Files Build Phase</samp>.
  * Click the disclosure triangle next to the new build phase.
  * Choose <samp>Frameworks</samp> from the Destination list.
  * Drag Sparkle.framework from the Project Navigator left sidebar to the list in the new <samp>Copy Files</samp> phase.
* In <samp>Build Settings</samp> tab set "<samp>Runpath Search Paths</samp>" to `@loader_path/../Frameworks` (for non-Xcode projects add the flags `-Wl,-rpath,@loader_path/../Frameworks`).
* If you have your own process for copying/packaging Sparkle make sure to preserve symlinks!

### 2. Set up a Sparkle updater object

These instructions are for regular .app bundles. If you want to update a non-app bundle, such as a Preference Pane or a plug-in, follow [step 2 for non-app bundles](/documentation/bundles/).

* Open up your MainMenu.nib.
* Choose <samp>View › Utilities › Object Library...</samp>
* Type "Object" in the search field under the object library (at the bottom of the right sidebar) and drag an Object into the left sidebar of the document editor.
* Select the Object that was just added.
* Choose <samp>View › Utilities › Identity Inspector</samp>.
* Type `SUUpdater` in the <samp>Class</samp> box of the <samp>Custom Class</samp> section in the inspector.
* If you'd like, make a "<samp>Check for Updates...</samp>" menu item in the application menu; set its target to the `SUUpdater` instance and its action to `checkForUpdates:`.

### 3. Segue for security concerns

Since Sparkle is downloading executable code to your users' systems, you must be very careful about security. To let Sparkle know that a downloaded update is not corrupted and came from you (instead of a malicious attacker), we recommend:

  * Serving the update over HTTPS (your app *will not update on macOS 10.11* unless you comply with Apple's [App Transport Security](/documentation/app-transport-security/) requirements),
  * code-signing the application via Apple's Developer ID program and/or,
  * code-signing the published update archive with a DSA [signature](//en.wikipedia.org/wiki/Digital_signature) matching a public DSA key included in your app.

If you are not serving updates over HTTPS and you are linking against the 10.11 SDK, **your updates will be blocked by macOS El Capitan**. You *must* comply with Apple's [App Transport Security](/documentation/app-transport-security/). In short, HTTP requests will be rejected by the system unless an exception is added within your app. App Transport Security has other specific requirements too, so please test updating your app on 10.11 even if you already are serving over HTTPS!

#### DSA signatures

We recommend signing updates with DSA signatures. This type of signature is most secure and supported for all types of Sparkle updates.

* Updates using [Installer package](/documentation/package-updates/) (`.pkg`) must be signed with DSA.
* [Binary Delta updates](/documentation/delta-updates/) must be signed with DSA.
* [Updates of preference panes and plugins](/documentation/bundles/) must be signed with DSA.

DSA signatures are optional for updates using regular app bundles that are signed with Apple code signing (Apple's Developer ID program or your own certificate), but we still recommended DSA signatures as a backup.

To sign update's archive with a DSA signature:

  * First, make yourself a pair of DSA keys. This needs to be done only once. Sparkle includes a tool to help: (from the Sparkle distribution root):<br />
  `./bin/generate_keys`
  * Back up your private key (dsa_priv.pem) and <strong>keep it safe.</strong> You don't want anyone else getting it, and if you lose it, you may not be able to issue any new updates.
  * Add your public key (dsa_pub.pem) to the Resources folder of your Xcode project.
  * Add an `SUPublicDSAKeyFile` key to your Info.plist; set its value to your public key's filename—unless you renamed it, this will be dsa_pub.pem.

#### Apple code signing

If you are code-signing your application via Apple's Developer ID program (which is recommended for macOS 10.8+), Sparkle will ensure the new version's author matches the old version's. Sparkle also performs basic (but not deep) validation for testing if the new application is archived/distributed correctly as you intended.

  * Note that embedding the Sparkle.framework into the bundle of a Developer ID application requires that you code-sign the framework with your Developer ID keys. Xcode should do this automatically if you let it "<samp>Code Sign on Copy</samp>" Sparkle's framework.
  * You can diagnose code signing problems with [RB App Checker app](//brockerhoff.net/RB/AppCheckerLite/) and by checking logs in the Console.app.

If you decide to both code-sign your application and include a public DSA key for signing your update archive, Sparkle allows issuing a new update that changes either your code signing certificate or your DSA keys. Note however this is a last resort and should *only* be done if you lose access to one of them.

### 4. Publish your appcast

Sparkle uses appcasts to get information about software updates. An appcast is an RSS feed with some extra information for Sparkle's purposes.

* Make a copy of the sample appcast included in the Sparkle distribution.
* Read the sample appcast to familiarize yourself with the format, then edit out all the items and add one for the new version of your app by following the instructions at [Publishing an update](/documentation/publishing/#publishing-an-update).
* Upload your appcast to a webserver.
* Add a `SUFeedURL` key to your Info.plist; set its value to the URL of your appcast. We [strongly encourage you to use HTTPS](/documentation/app-transport-security/) URLs for the appcast.
* Remember that your bundle must have a [properly formatted](/documentation/publishing/#publishing-an-update) `CFBundleVersion` key in your Info.plist.

### 5. Test Sparkle out

* Make sure the version specified for the update in your appcast is _greater than the `CFBundleVersion` of the app you're running_.
* Run your app, then quit, Sparkle doesn't ask the user about updates until the _second_ launch, in order to make your users' first-launch impression cleaner.
* Run your app again. The update process should proceed as expected.

### Next steps

That's it! You're done! You don't have to do any more. But you might want to:

* [Adjust Sparkle's settings and behavior](/documentation/customization/) for your product.
* [Add update settings](/documentation/preferences-ui/) to your preferences panel.
* [Add binary delta updates](/documentation/delta-updates/) to your application.
* Learn about [gathering anonymous statistics about your users' systems](/documentation/system-profiling/).
