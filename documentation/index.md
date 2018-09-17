---
layout: documentation
id: documentation
title: Documentation
---
## Basic Setup

If your app already has an older version of Sparkle, see [upgrading from previous versions](/documentation/upgrading/).

Note that Sparkle does [not yet support](//github.com/{{ site.github_username }}/Sparkle/issues/363) sandboxed applications.

### 1. Add the Sparkle framework to your project

If you use [CocoaPods](//cocoapods.org):

  * Add `pod 'Sparkle'` to your Podfile.
  * Add or uncomment `use_frameworks!` in your Podfile.
  * [Xcode 7 and older require more steps](/documentation/cocoapods/).

If you don't have CocoaPods, then add Sparkle manually:

* Get the [latest version](//github.com/{{ site.github_username }}/Sparkle/releases) of Sparkle.
* Link the Sparkle framework to your app target:
  * Drag `Sparkle.framework` into the <samp>Frameworks</samp> folder of your Xcode project.
  * Be sure to check the "Copy items into the destination group's folder" box in the sheet that appears.
  * Make sure the box is checked for your app's target in the sheet's <samp>Add to targets</samp> list.
* Make sure the framework is copied into your app bundle:
  * Click on your project in the Project Navigator.
  * Click your target in the project editor.
  * Click on the <samp>Build Phases</samp> tab.
  * Choose <samp>Editor › Add Build Phase › Add Copy Files Build Phase</samp>.
  * Click the disclosure triangle next to the new build phase.
  * Choose <samp>Frameworks</samp> from the Destination list.
  * Drag Sparkle.framework from the Project Navigator left sidebar to the list in the new <samp>Copy Files</samp> phase.
* In <samp>Build Settings</samp> tab set "<samp>Runpath Search Paths</samp>" to `@loader_path/../Frameworks` (for non-Xcode projects add the flags `-Wl,-rpath,@loader_path/../Frameworks`).
* If you have your own process for copying/packaging your app make sure it preserves symlinks!

### 2. Set up a Sparkle updater object

These instructions are for regular .app bundles. If you want to update a non-app bundle, such as a Preference Pane or a plug-in, follow [step 2 for non-app bundles](/documentation/bundles/).

* Open up your MainMenu.xib.
* Choose <samp>View › Utilities › Object Library...</samp>
* Type "Object" in the search field under the object library (at the bottom of the right sidebar) and drag an Object into the left sidebar of the document editor.
* Select the Object that was just added.
* Choose <samp>View › Utilities › Identity Inspector</samp>.
* Type `SUUpdater` in the <samp>Class</samp> box of the <samp>Custom Class</samp> section in the inspector.
* If you'd like, make a "<samp>Check for Updates...</samp>" menu item in the application menu; set its target to the `SUUpdater` instance and its action to `checkForUpdates:`.

### 3. Segue for security concerns

Since Sparkle is downloading executable code to your users' systems, you must be very careful about security. To let Sparkle know that a downloaded update is not corrupted and came from you (instead of a malicious attacker), we recommend:

  * Serve updates over HTTPS.
    * Your app *will not update on macOS 10.11 or later* unless you comply with Apple's [App Transport Security](/documentation/app-transport-security/) requirements. HTTP requests will be rejected by the system.
    * You can get free certificates from [Let's Encrypt](https://certbot.eff.org/), and test [server configuration](https://mozilla.github.io/server-side-tls/ssl-config-generator/) with [ssltest](https://www.ssllabs.com/ssltest/).
  * Sign the application via Apple's Developer ID program.
  * Sign the published update archive with Sparkle's EdDSA (ed25519) signature.
    * Updates using [Installer package](/documentation/package-updates/) (`.pkg`) *must* be signed with EdDSA.
    * [Binary Delta updates](/documentation/delta-updates/) *must* be signed with EdDSA.
    * [Updates of preference panes and plugins](/documentation/bundles/) *must* be signed with EdDSA.
    * EdDSA signatures are optional for updates using regular app bundles that are signed with Apple code signing (Apple's Developer ID program), but we still recommended EdDSA signatures as a backup.

#### EdDSA (ed25519) signatures

To prepare signing with EdDSA signatures:

  1. First, run `./bin/generate_keys` tool (from the Sparkle distribution root). This needs to be done only once. This tool will do two things:
    * It will generate a private key and save it in your login Keychain on your Mac. You don't need to do anything with it, but don't lose access to your Mac's Keychain. If you lose it, you may not be able to issue any new updates!
    * It will print your public key to embed into applications. Copy that key (it's a base64-encoded string). You can run `./bin/generate_keys` again to see your public key at any time.
  2. Add your public key to your app's `Info.plist` as a [`SUPublicEDKey`](/documentation/customization/) property.

Sparkle before version 1.21 used to use only older DSA signatures, which are now deprecated. They are still supported for updating old apps, and both DSA and EdDSA may be used together.

#### Apple code signing

If you are code-signing your application via Apple's Developer ID program, Sparkle will ensure the new version's author matches the old version's. Sparkle also performs basic (but not deep) validation for testing if the new application is archived/distributed correctly as you intended.

  * Note that embedding the `Sparkle.framework` into the bundle of a Developer ID application requires that you code-sign the framework with your Developer ID keys. Xcode should do this automatically if you let it "<samp>Code Sign on Copy</samp>" Sparkle's framework.
  * You can diagnose code signing problems with [RB App Checker app](//brockerhoff.net/RB/AppCheckerLite/) and by checking logs in the Console.app.

If you both code-sign your application and include a public EdDSA key for signing your update archive, Sparkle allows issuing a new update that changes either your code signing certificate or your EdDSA keys. Note however this is a last resort and should *only* be done if you lose access to one of them.

### 4. Distributing your App

If you distribute your app as a [Apple-certificate-signed disk image](https://developer.apple.com/library/content/technotes/tn2206/_index.html#//apple_ref/doc/uid/DTS40007919-CH1-TNTAG17) (DMG):

  * Add an `/Applications` symlink in your DMG, or otherwise encourage the user to copy the app out of it (due to [quarantine/translocation](https://lapcatsoftware.com/articles/app-translocation.html) the app can't be updated when it's launched straight from the disk).
  * Make sure the DMG is signed with a Developer ID and use macOS 10.11.5 or later to sign it (an older OS may not sign correctly). Signed DMG archives are backwards compatible.

If you distribute your app as a ZIP or a tar archive:

  * Encourage users to move the app to `/Applications` (e.g. use [LetsMove](https://github.com/potionfactory/LetsMove/)). Due to [quarantine/translocation](https://lapcatsoftware.com/articles/app-translocation.html) the system will not allow the app to update itself until it's moved.
  * Avoid placing your app inside another folder in your archive, because copying of the folder as a whole doesn't remove the quarantine.
  * Avoid putting more than just the single app in the archive.

Sparkle supports updating from DMG, ZIP archives, tarballs, and installer packages, so you can generally reuse the same archive for distribution of your app on your website as well as Sparkle updates.

For Sparkle, tarballs and ZIPs are fastest and most reliable. DMG are slowest. Installer packages should be used only if absolutely necessary (e.g. kernel extensions).

### 5. Publish your appcast

Sparkle uses appcasts to get information about software updates. An appcast is an RSS feed with some extra information for Sparkle's purposes.

  * Add a [`SUFeedURL`](/documentation/customization/) property to your `Info.plist`; set its value to a URL where your appcast will be hosted, e.g. `https://yourcompany.example.com/appcast.xml`. We [strongly encourage you to use HTTPS](/documentation/app-transport-security/) URLs for the appcast.
  * Remember that your app bundle must have a [properly formatted `CFBundleVersion`](/documentation/publishing/#publishing-an-update) key in your `Info.plist`.

If you update regular app bundles and you have set up EdDSA signatures, you can use a tool to generate appcasts automatically:

  1. Build your app and compress it (e.g. in a DMG/ZIP/tar.bz2 archive), and put the archive in a new folder. This folder will be used to store all your future updates.
  2. Run `generate_appcast` tool from Sparkle's distribution archive specifying the path to the folder with update archives. Allow it to access the Keychain if it asks for it (it's needed to generate signatueres in the appcast).

        ./bin/generate_appcast /path/to/your/updates_folder/

  3. The tool will generate the appcast file (using filename from [`SUFeedURL`](/documentation/customization/)) and also [`*.delta` update](/documentation/delta-updates/) files for faster incremental updates. Upload your archives, the delta updates, and the appcast to your server.

You can also create the appcast file manually (not recommended):

  * Make a copy of the sample appcast included in the Sparkle distribution.
  * Read the sample appcast to familiarize yourself with the format, then edit out all the items and add one for the new version of your app by following the instructions at [Publishing an update](/documentation/publishing/#publishing-an-update).

### 6. Test Sparkle out

* Use an older verison of your app, or if you don't have one yet, make one seem older by editing `Info.plist` and change `CFBundleVersion` to a lower version.
  * A genuine older version of the app is required to test [delta updates](/documentation/delta-updates/), because Sparkle will ignore the delta update if the app doesn't match update's checksum.
  * Editing `CFBundleVersion` of the latest version of the app is useful for testing the latest version of Sparkle framework.
* Run the app, then quit. Sparkle doesn't ask the user about updates until the _second_ launch, in order to make your users' first-launch impression cleaner.
* Run the app again. The update process should proceed as expected.

Update process will be logged to `Console.app`. If anything goes wrong, you should find detailed explanation in the log.

### Next steps

That's it! You're done! You don't have to do any more. But you might want to:

* [Adjust Sparkle's settings and behavior](/documentation/customization/) for your product.
* [Add update settings](/documentation/preferences-ui/) to your preferences panel.
* [Add binary delta updates](/documentation/delta-updates/) to your application.
* Learn about [gathering anonymous statistics about your users' systems](/documentation/system-profiling/).
