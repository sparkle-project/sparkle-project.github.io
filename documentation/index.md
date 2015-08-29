---
layout: documentation
id: documentation
title: Documentation
---
## Basic Setup

Follow these simple steps, and you'll have your app auto-updating ASAP. Note that Sparkle does [not yet support](https://github.com/{{ site.github_username }}/Sparkle/issues/363) sandboxed applications.

### 1. Add the Sparkle framework to your project

If you are using [CocoaPods](https://cocoapods.org), then follow these [alternate instructions](/documentation/cocoapods).

* First, we'll link the Sparkle framework to your target:
  * Drag Sparkle.framework into the Frameworks folder of your Xcode project.
  * Be sure to check the "copy items into the destination group's folder" box in the sheet that appears.
  * Make sure the box is checked for your app's target in the sheet's Add to targets list.
* Now we'll make sure the framework is copied into your app bundle:
  * Click on your project in the Project Navigator.
  * Click your target in the project editor.
  * Click on the Build Phases tab.
  * Choose Editor -> Add Build Phase -> Add Copy Files Build Phase.
  * Click the disclosure triangle next to the new build phase.
  * Choose Frameworks from the Destination list.
  * Drag Sparkle.framework from the Project Navigator left sidebar to the list in the new Copy Files phase.
* In Build Settings tab set "Other Linker Flags" to <code>-Wl,-rpath,@loader_path/../Frameworks</code>.
* If you have your own process for copying/packaging Sparkle make sure to preserve symlinks!

### 2. Set up a Sparkle updater object

* Open up your MainMenu.nib.
* Choose View -> Utilities -> Object Library...
* Type "Object" in the search field under the object library (at the bottom of the right sidebar) and drag an Object into the left sidebar of the document editor.
* Select the Object that was just added.
* Choose View -> Utilities -> Identity Inspector.
* Type <code>SUUpdater</code> in the Class box of the Custom Class section in the inspector.
* If you'd like, make a "Check for Updates..." menu item in the application menu; set its target to the SUUpdater instance and its action to <code>checkForUpdates:</code>.
* These instructions only work for .app bundles, because the SUUpdater instance instantiated in the nib will always be the <code>sharedUpdater</code>, which updates the hosting .app bundle. If you want to update a non-app bundle, such as a Preference Pane, see [bundles](/documentation/bundles) for alternative instructions.

### 3. Segue for security concerns

* Since Sparkle is downloading executable code to your users' systems, you must be very careful about security.
* To let Sparkle know that a downloaded update is not corrupted and came from you (instead of a malicious attacker), we recommend:
  * Serving the update over https
  * Code-signing the application via Apple's Developer ID program or with your own certificate
  * Specifying a [DSA signature](https://en.wikipedia.org/wiki/Digital_signature) of the SHA-1 hash of the published update archive and including a public DSA key inside your update

* While signing your archive using DSA signatures strengthens your update and may open up to additional features, they are not required to be used if the update is:
  * Served over https (serving over http without using DSA signatures is now deprecated)
  * Packaged as a regular app (i.e, not a preference pane, plug-in, package installer, or some other type)
  * Distributed without providing binary delta patches for more efficient updates

* If you are code-signing your application via Apple's Developer ID program or your own certificate, Sparkle will ensure the new version's author matches the old version's. Sparkle also performs basic (but not deep) validation for testing if the new application is archived/distributed correctly as you intended.
  * Note that embedding the Sparkle.framework into the bundle of a Developer ID application requires that you code-sign the framework with your Developer ID keys. Xcode should do this automatically if you let it "Code Sign on Copy" Sparkle's framework.
  * You can diagnose code signing problems with [RB App Checker app](http://brockerhoff.net/RB/AppCheckerLite/) and by checking logs in the Console.app.

* If you are specifying a DSA signature of your update's archive:
  * First, make yourself a pair of DSA keys; Sparkle includes a tool to help:
  * (from the Sparkle distribution root):<br />
  <code>./bin/generate_keys</code>
  * You can use the keys this tool generates to sign your updates.
  * Back up your private key (dsa_priv.pem) and <strong>keep it safe.</strong> You don't want anyone else getting it, and if you lose it, you may not be able to issue any new updates.
  * Add your public key (dsa_pub.pem) to the Resources folder of your Xcode project.
  * Add an <code>SUPublicDSAKeyFile</code> key to your Info.plist; set its value to your public key's filename—unless you renamed it, this will be dsa_pub.pem.

* If you decide to both code-sign your application and include a public DSA key for signing your update archive, Sparkle allows issuing a new update that changes either your code signing certificate or your DSA keys. Note however this is a last resort and should *only* be done if you lose access to one of them.

### 4. Publish your appcast

* Sparkle uses appcasts to get information about software updates.
* An appcast is an RSS feed with some extra information for Sparkle's purposes.
* Make a copy of the sample appcast included in the Sparkle distribution.
* Read the sample appcast to familiarize yourself with the format, then edit out all the items and add one for the new version of your app by following the instructions at [Publishing an update](/documentation/publishing#publishing-an-update).
* Upload your appcast to a webserver.
* Add a <code>SUFeedURL</code> key to your Info.plist; set its value to the URL of your appcast. We strongly encourage you to use HTTPS URLs for the appcast.
* Remember that your bundle must have a [properly formatted](/documentation/publishing#publishing-an-update) <code>CFBundleVersion</code> key in your Info.plist.

### 5. Test Sparkle out

* Make sure the version specified for the update in your appcast is _greater than the CFBundleVersion of the app you're running_.
* Run your app, then quit, Sparkle doesn't ask the user about updates until the _second_ launch, in order to make your users' first-launch impression cleaner.
* Run your app again. The update process should proceed as expected.

### Next steps

That's it! You're done! You don't have to do any more. But you might want to:

* [Adjust Sparkle's settings and behavior](/documentation/customization) for your product.
* [Add update settings](/documentation/preferences-ui) to your preferences panel.
* [Add delta updates](/documentation/delta-updates) to your application.
* Learn about [gathering anonymous statistics about your users' systems](/documentation/system-profiling).
