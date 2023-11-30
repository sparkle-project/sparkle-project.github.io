---
layout: documentation
id: documentation
title: Documentation
---
## Basic Setup

If your app already has an older version of Sparkle or you wish to migrate to Sparkle 2, please see [upgrading from previous versions](/documentation/upgrading/).

[Sandboxed applications](/documentation/sandboxing) are only supported in Sparkle 2.

### 1. Add the Sparkle framework to your project

If you use [Swift Package Manager](https://swift.org/package-manager/):

  * In your Xcode project: <samp>File</samp> › <samp>Add Packages…</samp>
  * Enter `https://github.com/sparkle-project/Sparkle` as the package repository URL
  * Choose the Package Options. The default options will let Xcode automatically update versions of Sparkle 2.

    From Xcode's project navigator, if you right click and show the `Sparkle` package in Finder, you will find Sparkle's tools to generate and sign updates in `../artifacts/sparkle/Sparkle/bin/` (in Finder you may need to go up one folder from `checkouts` via `Go › Enclosing Folder`).

If you use [CocoaPods](//cocoapods.org):

  * Add `pod 'Sparkle'` to your Podfile.
  * Add or uncomment `use_frameworks!` in your Podfile.

If you use [Carthage](https://github.com/Carthage/Carthage):
  * Add `binary "https://sparkle-project.org/Carthage/Sparkle.json"` to your Cartfile.
  * Run `carthage update`
  * Link the Sparkle framework to your app target:
    * Drag the built `Carthage/Build/Mac/Sparkle.framework` into your Xcode project.
    * Make sure the box is checked for your app's target in the sheet's <samp>Add to targets</samp> list.
  * Make sure the framework is copied into your app bundle:
    * Click on your project in the Project Navigator.
    * Click your target in the project editor.
    * Click on the <samp>General</samp> tab.
    * In <samp>Frameworks, Libraries, and Embedded Content</samp> section, change Sparkle.framework to <samp>Embed & Sign</samp>.

    Sparkle's tools to generate and sign updates are not included from Carthage and need to be grabbed from [our latest release](//github.com/{{ site.github_username }}/Sparkle/releases/latest).

    Sparkle only supports using a `binary` origin with Carthage because Carthage strips necessary code signing information when building the project from source.

If you want to add Sparkle manually:

* Get the [latest version](//github.com/{{ site.github_username }}/Sparkle/releases/latest) of Sparkle.
* Link the Sparkle framework to your app target:
  * Drag `Sparkle.framework` into your Xcode project.
  * Be sure to check the "Copy items into the destination group's folder" box in the sheet that appears.
  * Make sure the box is checked for your app's target in the sheet's <samp>Add to targets</samp> list.
* Make sure the framework is copied into your app bundle:
  * Click on your project in the Project Navigator.
  * Click your target in the project editor.
  * Click on the <samp>General</samp> tab.
  * In <samp>Frameworks, Libraries, and Embedded Content</samp> section, change Sparkle.framework to <samp>Embed & Sign</samp>.
* In <samp>Build Settings</samp> tab set "<samp>Runpath Search Paths</samp>" to `@loader_path/../Frameworks` (for non-Xcode projects add the flags `-Wl,-rpath,@loader_path/../Frameworks`). By default, recent versions of Xcode set this to `@executable_path/../Frameworks` which is already sufficient for regular applications.
* If you have your own process for copying/packaging your app make sure it preserves symlinks!

---

If you enable Library Validation, which is part of the Hardened Runtime and required for notarization, you will also need to either sign your application with an `Apple Development` certificate for development (requires being in Apple's developer program), or disable library validation for Debug configurations only. Otherwise, the system may not let your application load Sparkle if you attempt to sign to run locally via an ad-hoc signature. This is not an issue for distribution when you sign your application with a Developer ID certificate.

Sandboxed applications using Sparkle 2 require [additional setup](/documentation/sandboxing).

[Pre-releases](//github.com/{{ site.github_username }}/Sparkle/releases) when available are published on GitHub. They are also available in Swift Package Manager, CocoaPods, and Carthage too by specifying the pre-release version in your project's manifest.

A more nightly build from our repository can be downloaded from our [GitHub Actions page](https://github.com/sparkle-project/Sparkle/actions?query=event%3Apush+is%3Asuccess+branch%3A2.x) by selecting a recent workflow commit and downloading the `Sparkle-distribution*.tar.xz` artifact. Alternatively, you may clone Sparkle's repository with all its submodules, run `make release`, and extract the binaries in the resulting `Sparkle-*.tar.xz` (or `.bz2`) archive.

### 2. Set up a Sparkle updater object

These instructions are for regular .app bundles in Cocoa. If you want to use Sparkle from other UI toolkits such as SwiftUI or want to instantiate the updater yourself, please visit [our programmatic setup](/documentation/programmatic-setup). If you want to update a non-app bundle, such as a Preference Pane or a plug-in, follow [step 2 for non-app bundles](/documentation/bundles/).

* Open up your MainMenu.xib.
* Choose <samp>View › Show Library…</samp>
* Type "Object" in the search field under the object library and drag an Object into the left sidebar of the document editor.
* Select the Object that was just added.
* Choose <samp>View › Inspectors › Identity</samp>.
* Type `SPUStandardUpdaterController` in the <samp>Class</samp> box of the <samp>Custom Class</samp> section in the inspector.
* If you'd like, make a "<samp>Check for Updates…</samp>" menu item in the application menu; set its target to the `SPUStandardUpdaterController` instance and its action to `checkForUpdates:`.

If you are using Sparkle 1, you will need to use `SUUpdater` instead of `SPUStandardUpdaterController` in the above steps. In Sparkle 2, `SUUpdater` is a deprecated stub. While it is still functional for transitional purposes, new applications will want to migrate to `SPUStandardUpdaterController`.

That's it. No other API calls are required to start the updater and have it manage checking for updates automatically. If you intend to pursue additional updater APIs, please first check [API Expectations](/documentation/programmatic-setup#api-expectations) from our programmatic setup.

### 3. Segue for security concerns

Because Sparkle is downloading executable code to your users' systems, you must be very careful about security. To let Sparkle know that a downloaded update is not corrupted and came from you (instead of a malicious attacker), we recommend:

  * Serve updates over HTTPS.
    * Your app *will not update on macOS 10.11 or later* unless you comply with Apple's [App Transport Security](/documentation/app-transport-security/) requirements. HTTP requests will be rejected by the system.
    * You can get free certificates from [Let's Encrypt](https://certbot.eff.org/), and test [server configuration](https://mozilla.github.io/server-side-tls/ssl-config-generator/) with [ssltest](https://www.ssllabs.com/ssltest/).
  * Sign the application via Apple's Developer ID program.
  * Sign the published update archive with Sparkle's EdDSA (ed25519) signature.
    * Updates using [Installer package](/documentation/package-updates/) (`.pkg`) *must* be signed with EdDSA.
    * [Binary Delta updates](/documentation/delta-updates/) *must* be signed with EdDSA.
    * [Updates of preference panes and plugins](/documentation/bundles/) *must* be signed with EdDSA.
    * Updates to regular application bundles that are signed with Apple's Developer ID program are strongly recommended to be signed with EdDSA for better security and fallback. Sparkle now deprecates not using EdDSA for these updates.

Please ensure your signing keys are kept safe and cannot be stolen if your web server is compromised. One way to ensure this for example is not having your signing keys accessible from the machine that is hosting your product.

#### EdDSA (ed25519) signatures

To prepare signing with EdDSA signatures:

Run `./bin/generate_keys` tool (from the Sparkle distribution root). This needs to be done only once. This tool will do two things:

  * It will generate a private key and save it in your login Keychain on your Mac. You don't need to do anything with it, but do keep it safe. See further notes below if you happen to lose your private key.
  * It will print your public key to embed into applications. Copy that key (it's a base64-encoded string). You can run `./bin/generate_keys` again to see your public key at any time.

Then add your public key to your app's `Info.plist` as a [`SUPublicEDKey`](/documentation/customization/) property. Note that for new projects created with Xcode 12 or later, this file may be in the <samp>Info</samp> tab under your target settings.

Here is an example run of `./bin/generate_keys`:

```
A key has been generated and saved in your keychain. Add the `SUPublicEDKey` key to
the Info.plist of each app for which you intend to use Sparkle for distributing
updates. It should appear like this:

    <key>SUPublicEDKey</key>
    <string>pfIShU4dEXqPd5ObYNfDBiQWcXozk7estwzTnF9BamQ=</string>
```

You can use the `-x private-key-file` and `-f private-key-file` options to export and import the keys respectively when transferring keys to another Mac. Otherwise we recommend keeping the keys inside your Mac's keychain. Be sure to keep them safe and not lose them (they will be erased if your keychain or system is erased).

If your keys are lost however, you can still sign new updates for Developer ID signed applications through [key rotation](#rotating-signing-keys). Note this will not work for Installer package based updates or for applications that are not code signed. In those cases you may lose the ability to sign new updates.

Please visit [Migrating to EdDSA from DSA](eddsa-migration) if you are still providing DSA signatures so you can learn how to stop supporting them.

#### Apple code signing

If you are code-signing your application via Apple's Developer ID program, Sparkle will ensure the new version's author matches the old version's. Sparkle also performs shallow (but not deep) validation for testing if the new application's code signature is valid.
  * Note that embedding the `Sparkle.framework` into the bundle of a Developer ID application requires that you code-sign the framework and its helper tools with your Developer ID keys. Xcode should do this automatically if you create an archive via <samp>Product › Archive</samp> and <samp>Distribute App</samp> choosing <samp>Developer ID</samp> method of distribution.
  * You can diagnose code signing problems with `codesign --deep -vvv --verify <path-to-app>` for code signing validity, `spctl -a -t exec -vv <path-to-app>` for Gatekeeper validity, and by checking logs in the Console.app. See Apple's [Code Signing in Depth](https://developer.apple.com/library/archive/technotes/tn2206/_index.html) for more code signing details.

#### Rotating signing keys

For regular application updates, if you both code-sign your application with Apple's Developer ID program and include a public EdDSA key for signing your update archive, Sparkle allows rotating keys by issuing a new update that changes either your Apple code signing certificate or your EdDSA keys.

We recommend rotating keys only when necessary like if you need to change your Developer ID certificate, lose access to your EdDSA private key, or need to change (Ed)DSA keys due to [migrating away from DSA](eddsa-migration).

### 4. Distributing your App

We recommend distributing your app in Xcode by creating a <samp>Product › Archive</samp> and <samp>Distribute App</samp> choosing <samp>Developer ID</samp> method of distribution. Using Xcode's Archive Organizer will ensure Sparkle's helper tools are code signed properly for distribution. In automated environments, this can instead be done using `xcodebuild archive` and `xcodebuild -exportArchive`.

If you distribute your app on your website as a [Apple-certificate-signed disk image](https://developer.apple.com/library/content/technotes/tn2206/_index.html#//apple_ref/doc/uid/DTS40007919-CH1-TNTAG17) (DMG):

  * Add an `/Applications` symlink in your DMG, to encourage the user to copy the app out of it.
  * Make sure the DMG is signed with a Developer ID and use macOS 10.11.5 or later to sign it (an older OS may not sign correctly). Signed DMG archives are backwards compatible.

If you distribute your app on your website as a ZIP or a tar archive (due to [app translocation](https://lapcatsoftware.com/articles/app-translocation.html)):

  * Avoid placing your app inside another folder in your archive, because copying of the folder as a whole doesn't remove the quarantine.
  * Avoid putting more than just the single app in the archive.

If your app is running from a read-only mount, you can encourage (if you want) your user to move the app into /Applications. Some frameworks, although not officially sanctioned here, exist for this purpose. Note Sparkle will not by default automatically disturb your user if an update cannot be performed.

Sparkle supports updating from ZIP archives, tarballs, disk images (DMGs), and installer packages. While you can reuse the same archive for distribution of your app on your website, we recommend serving ZIPs or tarballs (e.g. tar.xz) for updates because they are the fastest and most reliable formats for Sparkle. [Installer packages](/documentation/package-updates/) should rarely be used for distribution or updates.

### 5. Publish your appcast

Sparkle uses appcasts to get information about software updates. An appcast is an RSS feed with some extra information for Sparkle's purposes.

  * Add a [`SUFeedURL`](/documentation/customization/) property to your `Info.plist`; set its value to a URL where your appcast will be hosted, e.g. `https://yourcompany.example.com/appcast.xml`. We [strongly encourage you to use HTTPS](/documentation/app-transport-security/) URLs for the appcast.
  * Note that your app bundle must have an incrementing and [properly formatted `CFBundleVersion`](https://developer.apple.com/documentation/bundleresources/information_property_list/cfbundleversion) key in your `Info.plist`. Sparkle uses this to compare and determine the latest version of your bundle.

If you update regular app bundles and you have set up EdDSA signatures, you can use a tool to generate appcasts automatically:

  1. Build your app and compress it (e.g. in a ZIP/tar.xz/DMG archive), and put the archive in a new folder. This folder will be used to store all your future updates.
  2. Run `generate_appcast` tool from Sparkle's distribution archive specifying the path to the folder with update archives. Allow it to access the Keychain if it asks for it (it's needed to generate signatures in the appcast).

        ./bin/generate_appcast /path/to/your/updates_folder/

  3. The tool will generate the appcast file (using filename from [`SUFeedURL`](/documentation/customization/)) and also [`*.delta` update](/documentation/delta-updates/) files for faster incremental updates. Upload your archives, the delta updates, and the appcast to your server.

When generating the appcast, if an `.html` file exists with the same name as the archive, then it will added as the `releaseNotesLink`. Run `generate_appcast -h` for a full overview and list of supported options.

You can also create the appcast file manually (not recommended):

  * Make a copy of the sample appcast included in the Sparkle distribution.
  * Read the sample appcast to familiarize yourself with the format, then edit out all the items and add one for the new version of your app by following the instructions at [Publishing an update](/documentation/publishing/#publishing-an-update).

### 6. Test Sparkle out

* Use an older version of your app, or if you don't have one yet, make one seem older by editing `Info.plist` and change `CFBundleVersion` to a lower version.
  * A genuine older version of the app is required to test [delta updates](/documentation/delta-updates/), because Sparkle will ignore the delta update if the app doesn't match update's checksum.
  * Editing `CFBundleVersion` of the latest development version of the app is useful for testing the latest version of Sparkle framework.
* Run the app, then quit. By default, Sparkle doesn't ask the user's permission for checking updates until the _second_ launch, in order to make your users' first-launch impression cleaner.
* Run the app again. The update process should proceed as expected. Note by default, Sparkle checks for updates in the background once every 24 hours. To test automatic update checks immediately, run `defaults delete my-bundle-id SULastCheckTime` to clear the last update check time before launching the app. Alternatively, initiate a manual update check from the app's menu bar.

The update process will be logged to `Console.app`. If anything goes wrong, you should find detailed explanation in the log.

Make sure to also keep Sparkle's debug symbols files (`.dSYM`) around as they will be useful for symbolicating crash logs if something were to go wrong.

### Next steps

That's it! You're done! You don't have to do any more. But you might want to:

* [Read more on publishing an update](/documentation/publishing/#publishing-an-update)
* [Customize Sparkle's settings and behavior](/documentation/customization/) for your product.
* [Add update settings](/documentation/preferences-ui/) to your settings panel.
* [Add binary delta updates](/documentation/delta-updates/) to your application.
* [Add gentle update reminders](/documentation/gentle-reminders) for your application.
* [Learn about gathering anonymous statistics about your users' systems](/documentation/system-profiling/).
* [Review Sparkle 2's API Reference](/documentation/api-reference)
