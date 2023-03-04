---
layout: documentation
id: documentation
title: Sandboxing with Sparkle
---

Using Sparkle in a sandboxed application is only supported in Sparkle 2. If you do not sandbox your application, you may skip this page unless you are interested in [Removing the XPC Services](#removing-xpc-services).

If you are migrating from an older beta version of Sparkle 2, the XPC Services have been moved into Sparkle's framework bundle. You will need to stop bundling the XPC Services directly inside your application and add Info.plist keys to enable the services. This is described in the sections below.

## Sandboxing

### Integration

In order for Sparkle to function in a sandboxed environment, the application must call out to XPC Services and ensure the application's entitlement requirements are met.

Sparkle by default bundles two XPC Services inside the framework for sandboxing your application:

* Installer.xpc (org.sparkle-project.InstallerLauncher.xpc prior to Sparkle 2.2)
* Downloader.xpc (org.sparkle-project.Downloader.xpc prior to Sparkle 2.2; optional)

Please read the below sections on full integration details.

#### Installation

The Installer XPC Service is required for Sandboxed applications. Sparkle by default bundles this XPC Service in its framework bundle.

This service lets Sparkle install updates outside of your application's sandbox.

To enable the service, you must set [SUEnableInstallerLauncherService](/documentation/customization#sandboxing-settings) boolean to `YES` in your application's Info.plist.

#### Communication

To allow Sparkle to communicate to its running installer tools, you will need to add the following temporary exception to your `.entitlements` file for your sandboxed application:

```xml
<key>com.apple.security.temporary-exception.mach-lookup.global-name</key>
<array>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)-spks</string>
    <string>$(PRODUCT_BUNDLE_IDENTIFIER)-spki</string>
</array>
```

If you are building your application in Xcode, `$(PRODUCT_BUNDLE_IDENTIFIER)` will be automatically substituted with your application's bundle identifier.

In the unusual case that you cannot add entitlements (e.g. your process inherits another application's restricted sandbox), you will need to [enable the Installer Connection & Status XPC Services](/documentation/customization#sandboxing-settings) instead and enable embedding the services in Sparkle's `ConfigCommon.xcconfig` when building Sparkle from source.

#### Downloader

The Downloader XPC Service is optional for Sandboxed applications. Sparkle by default bundles this XPC Service in its framework bundle.

Use this service only if your sandboxed application does not request the `com.apple.security.network.client` entitlement. The downloader service allows using Sparkle without forcing the network client entitlement on your entire application. There are a couple caveats with using the downloader service though:

* It may not work well if your release notes reference external content that would require making additional network requests.
* We fall back to using the legacy WebKit WebView for release notes due to a [known WKWebView defect](https://github.com/feedback-assistant/reports/issues/1). Please file a feedback report to Apple duping to FB6993802 preventing the use of WKWebView without enforcing the `com.apple.security.network.client` entitlement on your sandboxed application.

To enable the service, you must set [SUEnableDownloaderService](/documentation/customization#sandboxing-settings) boolean to `YES` in your application's Info.plist.

### Code Signing

If you follow standard workflows and archive & export your application to [Distribute your App](/documentation#4-distributing-your-app), which we recommend, you do not need to especially do anything for signing Sparkle or its XPC Services and may skip this section.

However, if you need to code sign Sparkle with a specific certificate for development or use an alternative workflow for distributing your application outside of Xcode's archive and export workflow, then you will need to manually re-sign Sparkle and its XPC Services with your own certificate.

By default, Sparkle distributions include XPC Services and helper tools that are signed with an ad-hoc signature, Hardened Runtime enabled, and a `com.apple.security.get-task-allow` entitlement to allow debugging. This combination works for common development workflows, but is not ideal for distribution.

If you `Code Sign on Copy` Sparkle.framework, Xcode will re-sign Sparkle with your project's certificate but will not re-sign the XPC Services and other helpers inside the framework. Xcode does re-sign these services and helpers, preserves the Hardened Runtime, and strips the `com.apple.security.get-task-allow` entitlements when you Archive and Export an application for notarization and thus works sufficiently well there however.

You may re-sign Sparkle for distribution manually if needed like so:

```sh
codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime Sparkle.framework/Versions/B/XPCServices/Installer.xpc
codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime --entitlements Entitlements/Downloader.entitlements Sparkle.framework/Versions/B/XPCServices/Downloader.xpc

codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime Sparkle.framework/Versions/B/Autoupdate
codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime Sparkle.framework/Versions/B/Updater.app

codesign -f -s "$CODE_SIGN_IDENTITY" -o runtime Sparkle.framework
```

Adjust the paths and code sign identity accordingly. The `--preserve-metadata=entitlements` is not used because we don't want to preserve the existing `com.apple.security.get-task-allow` entitlements for distribution. Also the `--deep` option is not used here because the Downloader XPC Service needs to be signed with a specific entitlement that aren't applicable to the other binaries.

Due to different code signing requirements, please do not add `--deep` to `OTHER_CODE_SIGN_FLAGS` or from custom build scripts when signing your application. This is a common source of Sandboxing errors.

### Removing XPC Services

If you do not sandbox your application, we do not recommend using Sparkle's XPC Services. You may choose to remove Sparkle's XPC Services in a post install script when copying the framework to your application. Alternatively you can alter Sparkle's `ConfigCommon.xcconfig` to not embed the XPC Services. This is optional and up to you. The same applies if you do sandbox your application but do not need to use or embed the Downloader XPC Service in particular.

### Testing

If Xcode has issues running your application using Sparkle and its XPC Services (such as being unable to attach to the process), try editing your project's Scheme and disable *Debug XPC services used by app* or test your application detached from Xcode to see if it works there. This shouldn't be an issue in a development environment if you follow the standard workflow we suggest, but this may be needed if the XPC Service is signed with the Hardened Runtime and not with the `com.apple.security.get-task-allow` entitlement.
