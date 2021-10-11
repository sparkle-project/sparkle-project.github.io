---
layout: documentation
id: documentation
title: Sandboxing with Sparkle
---

Using Sparkle in a sandboxed application is only supported in Sparkle 2.0, which is currently in beta. If you do not sandbox your application, you may skip this page.

If you are migrating from an older version of Sparkle 2 beta, the XPC Services have been moved into Sparkle's framework bundle. You will need to stop bundling the XPC Services directly inside your application and add Info.plist keys to enable the services now. This is described in the sections below.

## Sandboxing

### XPC Services

In order for Sparkle to work in a sandboxed application, the framework must call out to XPC Services to perform updating and installation.

Sparkle by default bundles two XPC Services inside the framework for sandboxing:

* org.sparkle-project.InstallerLauncher.xpc
* org.sparkle-project.Downloader.xpc

There are two other XPC Services, not bundled by default, for communicating to Sparkle's installer helpers:

* org.sparkle-project.InstallerConnection.xpc
* org.sparkle-project.InstallerStatus.xpc

In order for Sparkle to work in a sandboxed application, the application must call out to XPC Services to perform the updating and installation. Only the InstallerLauncher XPC Service is strictly required. The other serices are optional depending on your use case.

If you build Sparkle yourself, you can optionally choose to change `XPC_SERVICE_BUNDLE_ID_PREFIX` in `ConfigCommon.xcconfig` from `org.sparkle-project` to your own prefix. In this configuration file, you can also choose which services Sparkle should bundle (by setting `SPARKLE_EMBED_*_XPC_SERVICE` variables). Please see notes below on integrating each of these services.

#### Installer Launcher Service

The Installer Launcher Service is required for Sandboxed applications. Sparkle by default bundles this XPC Service in its framework bundle.

To enable the service, you must set [SUEnableInstallerLauncherService](/documentation/customization#sandboxing-settings) boolean to `YES` in your application's Info.plist.

#### Downloader Service

The Downloader XPC Service is optional for Sandboxed applications. Sparkle by default bundles this XPC Service in its framework bundle.

Use this service only if your sandboxed application does not request the `com.apple.security.network.client` entitlement. The downloader service allows using Sparkle without forcing the network client entitlement on your entire application. There are a couple caveats with using the downloader service though:

* It may not work well if your release notes reference external content that would require making additional network requests.
* We fall back to using the legacy WebKit WebView for release notes due to a [known WKWebView defect](https://github.com/feedback-assistant/reports/issues/1). Please file a feedback report to Apple duping to FB6993802 if you want Sparkle's downloader XPC Service to use WKWebView without the `com.apple.security.network.client` entitlement.

To enable the service, you must set [SUEnableDownloaderService](/documentation/customization#sandboxing-settings) boolean to `YES` in your application's Info.plist.

#### Installer Connection & Status Services

The Installer Connection & Status Services are optional (as of changes integrated on [April 4, 2021](https://github.com/sparkle-project/Sparkle/pull/1812)) and are not bundled in the Sparkle framework by default. Instead of using these services, we recommend adding the following entitlement to your sandboxed application:

```xml
<key>com.apple.security.temporary-exception.mach-lookup.global-name</key>
<array>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)-spks</string>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)-spki</string>
</array>
```

This entitlement allows Sparkle to communicate with its installer and updater progress tools on your system. If you are building your application in Xcode, `$(PRODUCT_BUNDLE_IDENTIFIER)` will be automatically substituted with your application's bundle identifier.

If you cannot add entitlements (eg: your process inherits another application's restricted sandbox), you will need to [enable the XPC Services](/documentation/customization#sandboxing-settings) instead and need to enable embedding the services in Sparkle's `ConfigCommon.xcconfig`.

### Code Signing

If you follow standard workflows and use Xcode's Archive Organizer to [Distribute your App](/documentation#4-distributing-your-app), which we recommend, you do not need to especially do anything for signing Sparkle's XPC Services and may skip this section.

However, if you need to code sign Sparkle's XPC Services with a specific certificate for development or use an alternative workflow for distributing your application outside of Xcode's Archive Organizer, you will need to manually re-sign Sparkle's XPC Services with your own certificate.

By default, Sparkle distributions include XPC Services that are signed with an ad-hoc signature and Hardened Runtime enabled. This combination works for common development workflows.

If you `Code Sign on Copy` Sparkle.framework, Xcode will re-sign Sparkle with your project's certificate but will not re-sign the XPC Services inside the framework. Xcode does re-sign these services and preserves the Hardened Runtime when you Archive an application for distribution and thus suffices there however.

You may re-sign Sparkle's XPC Services manually if needed like so:

```sh
codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime Sparkle.framework/Versions/B/XPCServices/org.sparkle-project.InstallerLauncher.xpc
codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime --preserve-metadata=entitlements Sparkle.framework/Versions/B/XPCServices/org.sparkle-project.Downloader.xpc
```

Adjust the paths and code sign identity accordingly.

### Removing XPC Services

If you do not sandbox your application, we do not recommend using Sparkle's XPC Services. You may choose to remove Sparkle's XPC Services in a post install script when copying the framework to your application. Alternatively you can alter Sparkle's `ConfigCommon.xcconfig` to not embed the XPC Services. This is optional and up to you. The same applies if you do sandbox your application but do not need to use or embed the Downloader XPC Service in particular.

### Testing

Most likely Xcode will not have an issue with running your application using Sparkle and its XPC Services. But if Xcode chokes, try editing your project's Scheme and disable *Debug XPC services used by app* or test your application detatched from Xcode to see if it works there.
