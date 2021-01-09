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

### Downloader Service

The last downloader XPC Service is optional. Use it only if your sandboxed application does not request the `com.apple.security.network.client` entitlement. The downloader service allows using Sparkle without forcing the network client entitlement on your entire application. There are a couple caveats with using the downloader service though:

* It may not work well if your release notes reference external content that would require making additional network requests.
* We fall back to using legacy WebKit view due to a [known WKWebView defect](https://github.com/feedback-assistant/reports/issues/1).

### Code Signing

All the other XPC Services are required. You will also need to code sign these services by running:

```
./bin/codesign_embedded_executable "Developer ID Application" XPCServices/*.xpc
```

I used "Developer ID Application" for my certificate; you may need to adjust this.

### Adding the Services

Then you will need to add the XPC Services to your application project:

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

Due to the `./bin/codesign_embedded_executable` script signing the XPC Services with the Hardened Runtime enabled, Xcode cannot debug the XPC Services and you may see that updating does not work when your application is attached to Xcode. You can work around this either by editing your project's Scheme and disabling *Debug XPC services used by app*, or by testing your application detached from Xcode, or by altering the script to not sign the services with Hardened Runtime enabled for development builds.

