---
layout: documentation
id: documentation
title: Sandboxing with Sparkle
---

Note using Sparkle in a sandboxed application is only supported in Sparkle 2.0, which is currently in beta.

## Sandboxing

### XPC Services

In order for Sparkle to work in a sandboxed application, the application must call out to XPC services to perform the updating and installation. Note if you do not sandbox your application, you do not need to use any XPC services and may skip this page.

In an extracted `Sparkle-2.0.0.tar.xz` distribution in the `XPCServices/` directory you will notice:

* org.sparkle-project.InstallerConnection.xpc
* org.sparkle-project.InstallerLauncher.xpc
* org.sparkle-project.InstallerStatus.xpc
* org.sparkle-project.Downloader.xpc & org.sparkle-project.Downloader.entitlements

If you build Sparkle yourself, you can optionally choose to change `XPC_SERVICE_BUNDLE_ID_PREFIX` in `ConfigCommon.xcconfig` from `org.sparkle-project` to your own prefix.

### Downloader Service

The last downloader XPC Service is optional. Use it only if your sandboxed application does not request the `com.apple.security.network.client` entitlement. The downloader service allows using Sparkle without forcing the network client entitlement on your entire application. There are a couple caveats with using the downloader service though:

* It may not work well if your release notes reference external content that would require making additional network requests.
* We fall back to using legacy WebKit view for release notes due to a [known WKWebView defect](https://github.com/feedback-assistant/reports/issues/1).

### Code Signing

**Note**: By default, Sparkle builds XPC Services with an ad-hoc signature and with the Hardened Runtime enabled. In many cases this may be suffice for development. If you use Xcode's Archive Organizer to [Distribute your App](/documentation#4-distributing-your-app), it will manage re-signing these services with a Developer ID certificate and preserve the applied entitlements automatically.

Otherwise if you use alternate methods of distributing your application or you need to use a different certificate for development, you can code sign these services by running:

```
./bin/codesign_embedded_executable "Developer ID Application" XPCServices/*.xpc
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

