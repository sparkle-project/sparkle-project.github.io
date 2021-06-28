---
layout: documentation
id: documentation
title: Sandboxing with Sparkle
---

Note using Sparkle in a sandboxed application is only supported in Sparkle 2.0, which is currently in beta. If you do not sandbox your application, you should not use any of the XPC Services and may skip this page.

## Sandboxing

### XPC Services

In order for Sparkle to work in a sandboxed application, the application must call out to XPC Services to perform the updating and installation. Only the InstallerLauncher XPC Service is strictly required. The other serices are optional depending on your use case.

In an extracted `Sparkle-2.0.0.tar.xz` distribution in the `XPCServices/` directory you will notice:

* org.sparkle-project.InstallerLauncher.xpc
* org.sparkle-project.Downloader.xpc & org.sparkle-project.Downloader.entitlements
* org.sparkle-project.InstallerConnection.xpc & org.sparkle-project.InstallerConnection.entitlements
* org.sparkle-project.InstallerStatus.xpc & org.sparkle-project.InstallerStatus.entitlements

If you build Sparkle yourself, you can optionally choose to change `XPC_SERVICE_BUNDLE_ID_PREFIX` in `ConfigCommon.xcconfig` from `org.sparkle-project` to your own prefix. Please see notes below on integrating each of these services.

#### Installer Launcher Service

The Installer Launcher Service is required. This service bundles a copy of Sparkle's helper tools `Autoupdate` and `Updater.app`.

To optimize for space, a sandboxed application that uses this service may optionally choose to remove the helper tools from the `Sparkle.framework` and re-sign the framework:

```
rm -f Sparkle.framework/Autoupdate
rm -f Sparkle.framework/Updater
rm -f Sparkle.framework/Versions/A/Autoupdate
rm -rf Sparkle.framework/Versions/A/Updater.app
codesign -f -s "-" Sparkle.framework # re-signing the framework with an ad-hoc signature here
codesign --verify --deep Sparkle.framework # to check the framework is signed correctly
```

#### Downloader Service

The Downloader XPC Service is optional. Use it only if your sandboxed application does not request the `com.apple.security.network.client` entitlement. The downloader service allows using Sparkle without forcing the network client entitlement on your entire application. There are a couple caveats with using the downloader service though:

* It may not work well if your release notes reference external content that would require making additional network requests.
* We fall back to using legacy WebKit view for release notes due to a [known WKWebView defect](https://github.com/feedback-assistant/reports/issues/1).

#### Installer Connection & Status Services

The Installer Connection & Status Services are optional (as of changes integrated on [April 4, 2021](https://github.com/sparkle-project/Sparkle/pull/1812)). Instead of using these services, we recommend adding the following entitlement to your sandboxed application:

```
<key>com.apple.security.temporary-exception.mach-lookup.global-name</key>
<array>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)-spks</string>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)-spki</string>
</array>
```

This entitlement allows Sparkle to communicate with its installer and updater progress tools on your system. If you are building your application in Xcode, `$(PRODUCT_BUNDLE_IDENTIFIER)` will be automatically substituted with your application's bundle identifier.

If you cannot add entitlements (eg: your process inherits another application's sandbox), you will need to use the XPC Services instead.

### Code Signing

If you can:
* Use the distributed XPC Services that are signed with an ad-hoc signature and Hardened Runtime enabled for development
* Use Xcode's Archive Organizer to [Distribute your App](/documentation#4-distributing-your-app), which will re-sign your XPC Services with a Developer ID certificate and preserve entitlements / hardened runtime during export
* Avoid using the Installer Connection & Status Services above, which both need entitlements targetting your application's bundle identifier

Then you can probably skip onto [Adding the XPC Services](#adding-the-xpc-services) section.

Otherwise if you use alternate methods of distributing your application, or you need to use a different certificate for development, you can code sign these services by running the `bin/codesign_xpc_service` script. For example:

```
./bin/codesign_xpc_service "Developer ID Application" XPCServices/org.sparkle-project.InstallerLauncher.xpc XPCServices/org.sparkle-project.Downloader.xpc
```

I used "Developer ID Application" for my certificate; you may need to adjust this.

### Adding the XPC Services

* Add the XPC Services to your app target:
  * Drag the XPC Services you need into your Xcode project.
  * Be sure to check the “Copy items into the destination group’s folder” box in the sheet that appears.
  * Make sure the box is checked for your app’s target in the sheet’s Add to targets list
* Make sure the XPC Services are properly copied in your app bundle:
  * Click on your project in the Project Navigator.
  * Click your target in the project editor.
  * Click on the Build Phases tab.
  * Remove the XPC Services in the <samp>Copy Bundle Resources</samp> phase if Xcode auto-added them there.
  * Click <samp>+</samp> to add a new Copy Files Phase.
  * Choose <samp>XPC Services</samp> as the Destination.
  * Drag the XPC Services you added from Xcode's project navigator to the new Copy Files Phase.

### Testing

Due to the XPC Services being code signed with the Hardened Runtime enabled by default, Xcode cannot debug the XPC Services and you may see that updating does not work when your application is attached to Xcode's debugger. You can work around this either by editing your project's Scheme and disabling *Debug XPC services used by app* or by testing your application detached from Xcode.
