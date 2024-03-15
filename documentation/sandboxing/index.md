---
layout: documentation
id: documentation
title: Sandboxing with Sparkle
---

This guide shows how to use Sparkle 2 in sandboxed applications. If you do not sandbox your application, you should skip this guide unless you are interested in [Removing the XPC Services](#removing-xpc-services). If you choose to later disable application sandboxing, you should also undo enabling the services and entitlements mentioned in this guide.

If you are migrating from using an older beta version of Sparkle 2 in a sandboxed app, the XPC Services have been moved into Sparkle's framework bundle. You will need to stop bundling the XPC Services directly inside your application and add Info.plist keys to enable the services. This is described in the sections below.

Please note that Sparkle 1 does not support sandboxed applications.

## Sandboxing

### Integration

In order for Sparkle to function in a sandboxed environment, the application must call out to XPC Services and ensure the application's entitlement requirements are met.

Sparkle by default bundles two XPC Services inside the framework for sandboxing your application:

* Installer.xpc (org.sparkle-project.InstallerLauncher.xpc prior to Sparkle 2.2)
* Downloader.xpc (org.sparkle-project.Downloader.xpc prior to Sparkle 2.2; for apps that don't have network client access)

Please read the below sections on full integration details.

#### Installation

The Installer XPC Service is required for Sandboxed applications.

This service lets Sparkle install updates outside of your application's sandbox.

To enable the service, you must set [SUEnableInstallerLauncherService](/documentation/customization#sandboxing-settings) boolean to `YES` in your application's Info.plist.

#### Communication

To allow Sparkle to communicate to its running installer tools for sandboxed applications, you will need to add the following temporary exception to your `.entitlements` file:

```xml
<key>com.apple.security.temporary-exception.mach-lookup.global-name</key>
<array>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)-spks</string>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)-spki</string>
</array>
```

If you are building your application in Xcode, `$(PRODUCT_BUNDLE_IDENTIFIER)` will be automatically substituted with your application's bundle identifier.

In the rare case that you cannot add entitlements (e.g. your process inherits another application's restricted sandbox), you will need to [enable the Installer Connection & Status XPC Services](/documentation/customization#sandboxing-settings) instead and enable embedding the services in Sparkle's `ConfigCommon.xcconfig` when building Sparkle from source.

#### Downloader

The Downloader XPC Service is only needed for Sandboxed applications that do not request the `com.apple.security.network.client` entitlement. If your sandboxed application already needs to use this entitlement for network access, then you should not enable the Downloader XPC Service.

The downloader service allows using Sparkle without forcing the network client entitlement on your entire sandboxed application. There are a few drawbacks with using the downloader service though which you may want to consider:

* We fall back to using WebKit's deprecated `WebView` for release notes due to a [known WKWebView defect](https://github.com/feedback-assistant/reports/issues/1). Please file a bug report on [WebKit's issues tracker](https://webkit.org/reporting-bugs/) or a Feedback Assistant report to Apple relating to FB6993802 preventing the use of WKWebView without having the `com.apple.security.network.client` entitlement on your sandboxed application. Emphasize that this defect blocks your app's adoption from WebKit1 to WebKit2.
* [Adapting release notes based on the currently installed version](/documentation/publishing#adapting-release-notes-based-on-currently-installed-version) is not supported because this feature is not implemented for Sparkle's legacy WebKit view.
* It may not work well if your release notes reference external content that would require making additional network requests.
* As of Sparkle 2.6, the Downloader XPC Service is not sandboxed by default. If you want to sandbox this service, you will need to uncomment `DOWNLOADER_SANDBOXED_ENTITLEMENTS` and modify `XPC_SERVICE_BUNDLE_ID_PREFIX` in Sparkle's `ConfigCommon.xcconfig` when building Sparkle from source. Sandboxing this XPC Service requires using a custom bundle ID for it, otherwise conflicts may arise with other sandboxed apps using the service.

To enable the service, you must set [SUEnableDownloaderService](/documentation/customization#sandboxing-settings) boolean to `YES` in your application's Info.plist.

### Code Signing

If you follow standard workflows and archive & export your application to [Distribute your App](/documentation#4-distributing-your-app), which we recommend, you do not need to especially do anything for signing Sparkle or its XPC Services and may skip this section.

However, if you need to code sign Sparkle with a specific certificate for development or use an alternative workflow for distributing your application outside of Xcode's archive and export workflow, then you will need to manually re-sign Sparkle and its XPC Services with your own certificate.

By default, Sparkle distributions include XPC Services and helper tools that are signed with an ad-hoc signature and Hardened Runtime enabled. Older Sparkle distributions (before 2.4) may also include a `com.apple.security.get-task-allow` entitlement. This combination works for common development workflows, but is not ideal for distribution.

If you `Code Sign on Copy` Sparkle.framework, Xcode will re-sign Sparkle with your project's certificate but will not re-sign the XPC Services and other helpers inside the framework. Xcode does re-sign these services and helpers, preserves the Hardened Runtime, and strips `com.apple.security.get-task-allow` entitlements when you Archive and Export an application for notarization and thus works sufficiently well there however.

You may re-sign Sparkle for distribution manually if needed like so:

```sh
codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime Sparkle.framework/Versions/B/XPCServices/Installer.xpc

# For Sparkle versions >= 2.6
codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime --preserve-metadata=entitlements Sparkle.framework/Versions/B/XPCServices/Downloader.xpc

# For Sparkle versions < 2.6
#codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime --entitlements Entitlements/Downloader.entitlements Sparkle.framework/Versions/B/XPCServices/Downloader.xpc

codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime Sparkle.framework/Versions/B/Autoupdate
codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime Sparkle.framework/Versions/B/Updater.app

codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime Sparkle.framework
```

Adjust the paths and code sign identity accordingly. The `--deep` option is not used here because the Downloader XPC Service may optionally be signed with a specific entitlement that aren't applicable to the other binaries.

Due to different code signing requirements, please do not add `--deep` to `OTHER_CODE_SIGN_FLAGS` or from custom build scripts when signing your application. This is a common source of Sandboxing errors.

### Testing

If Xcode has issues running your application using Sparkle and its XPC Services (such as being unable to attach to the process), try editing your project's Scheme and disable *Debug XPC services used by app* or test your application detached from Xcode to see if it works there. This may be needed because the XPC Services we distribute are signed with the Hardened Runtime which prevents debugging.

### Removing XPC Services

This section is optional and is for developers that want to trim down Sparkle.

If you do not sandbox your application and thus do not enable Sparkle's XPC Services, you may choose to remove these services in a post install script when copying the framework to your application. Alternatively you can alter Sparkle's `ConfigCommon.xcconfig` to not embed the XPC Services when building Sparkle from source.

The same can apply if you do sandbox your application but do not need to enable or embed the Downloader XPC Service in particular.
