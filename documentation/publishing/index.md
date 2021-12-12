---
layout: documentation
id: documentation
title: Publishing an update
---
## Publishing an update

So you're ready to release a new version of your app. How do you go about doing that?

### Archive your app

Put a copy of your .app (with the same name as the version it's replacing) in a .zip, .tar.gz, or .tar.bz2. If you distribute your .app in a .dmg, do _not_ zip up the .dmg.

Make sure symlinks are preserved when you create the archive. macOS frameworks use symlinks, and their code signature will be broken if your archival tool follows symlinks instead of archiving them. For creating zip archives, `ditto` can be used (behaves the same as Finder's Compress option):

```sh
ditto -c -k --sequesterRsrc --keepParent <src_path_to_app> <zip_dest>
```

If you can't use regular app bundles, you can also create an Installer .pkg with the same name as your app and put that .pkg in one of the aforementioned archive formats. By default Sparkle launches Installer without a GUI. If instead of .pkg extension you use .sparkle_interactive.pkg, then installation will run with a GUI and ask user to confirm every step.

### Secure your update

In order to prevent corruption and man-in-the-middle attacks against your users, you must cryptographically sign your updates.

Signatures are automatically generated when you make an appcast using `generate_appcast` tool. This is a recommended method.

To manually generate signatures for your updates, Sparkle includes a tool to help you make a EdDSA signature of the archive. From the Sparkle distribution:

```sh
./bin/sign_update path_to_your_update.zip
```

The output will be an XML fragment with your update's EdDSA signature and (optional) file size; you'll add this attribute to your enclosure in the next step.

Since 10.11, macOS has [App Transport Security](//developer.apple.com/library/prerelease/mac/technotes/App-Transport-Security-Technote/) policy which blocks apps from using insecure HTTP connections. This restriction applies to Sparkle as well, so you will need to serve your appcast and the update files over HTTPS.

### Update your appcast

You need to create an `<item>` for your update in your appcast. See the [sample appcast](/files/sparkletestcast.xml) for an example. Here's a template you might use:

```xml
<item>
    <title>Version 2.0 (2 bugs fixed; 3 new features)</title>
    <link>https://myproductwebsite.com</link>
    <sparkle:version>2.0</sparkle:version>
    <sparkle:releaseNotesLink>
        https://example.com/release_notes/app_2.0.html
    </sparkle:releaseNotesLink>
    <pubDate>Mon, 05 Oct 2015 19:20:11 +0000</pubDate>
    <enclosure url="https://example.com/downloads/app.zip.or.dmg.or.tar.etc"
                sparkle:edSignature="7cLALFUHSwvEJWSkV8aMreoBe4fhRa4FncC5NoThKxwThL6FDR7hTiPJh1fo2uagnPogisnQsgFgq6mGkt2RBw=="
                length="1623481"
                type="application/octet-stream" />
</item>
```

Test your update, and you're done!

Note on `sparkle:version`: Our previous documentation used to recommend specifying `sparkle:version` (and `sparkle:shortVersionString`) as part of the `enclosure` item. While this works fine, for overall consistency we now recommend specifying them as top level items instead as shown here. This is supported across all versions of Sparkle.

## Delta updates

If your app is large, or if you're updating primarily only a small part of it, you may find [delta updates](/documentation/delta-updates/) useful: they allow your users to only download the bits that have changed. The `generate_appcast` tool automatically generates delta updates.

## Internal build numbers

If you use internal build numbers for your `CFBundleVersion` key and a human-readable `CFBundleShortVersionString`, you can make Sparkle hide the internal version from your users.

Set the `sparkle:version` element in your item to the internal, machine-readable version (ie: "1248"). Then set a `sparkle:shortVersionString` element on in your item to the human-readable version (ie: "1.5.1"). For example:

```xml
<item>
    <title>Version 1.5.1 (bug fixes)</title>
    <link>https://myproductwebsite.com</link>
    <sparkle:version>1248</sparkle:version>
    <sparkle:shortVersionString>1.5.1</sparkle:shortVersionString>
    <sparkle:releaseNotesLink>
        https://example.com/release_notes/app_2.0.html
    </sparkle:releaseNotesLink>
    <pubDate>Mon, 05 Oct 2015 19:20:11 +0000</pubDate>
    <enclosure url="https://example.com/downloads/app.zip.or.dmg.or.tar.etc"
                sparkle:edSignature="7cLALFUHSwvEJWSkV8aMreoBe4fhRa4FncC5NoThKxwThL6FDR7hTiPJh1fo2uagnPogisnQsgFgq6mGkt2RBw=="
                length="1623481"
                type="application/octet-stream" />
</item>
```

Note that the internal version number (`CFBundleVersion` and `sparkle:version`) is intended to be machine-readable and is not generally suitable for formatted text or git changeset IDs.

## Minimum system version requirements

If an update to your application raises the required version of macOS, you can only offer that update to qualified users.

Add a `sparkle:minimumSystemVersion` child to the `<item>` in question specifying the required system version, such as "10.11.0" (be sure to specify a three-part version in form of *major.minor.patch*):

```xml
<item>
    <title>Version 2.0 (2 bugs fixed; 3 new features)</title>
    <link>https://myproductwebsite.com</link>
    <sparkle:version>2.0</sparkle:version>
    <sparkle:minimumSystemVersion>10.11.0</sparkle:minimumSystemVersion>
</item>
```

Note that Sparkle 2 only works with macOS 10.11 or later (macOS 10.9 or later for Sparkle 1), so that's the lowest minimum version you can use.

Sparkle also supports a `sparkle:maximumSystemVersion` element that can limit the maximum system version similarly.

Please note that if your application is built using the macOS 10.15 SDK or earlier, the system may report its operating system as 10.16.0 for [compatibility reasons](https://eclecticlight.co/2020/07/21/big-sur-is-both-10-16-and-11-0-its-official/). To minimize issues, we recommend building your application with an up to date Xcode and SDK.

Additionally in Sparkle 2, if the user checks for new updates manually and the cannot update because of an operating system requirement, the standard updater alert will inform the user their operating system is incompatible and provide them an option to visit your website specified by the `<link>` element.

## Major upgrades

If an update to your application is a major or paid upgrade, you may want to prevent the update from being installed automatically.

Add a `sparkle:minimumAutoupdateVersion` child to the `<item>` in question specifying the major update's `<CFBundleVersion>`, such as "2.0":

```xml
<item>
    <title>Version 2.1 (2 bugs fixed; 3 new features)</title>
    <link>https://myproductwebsite.com</link>
    <sparkle:version>2.1</sparkle:version>
    <sparkle:minimumAutoupdateVersion>2.0</sparkle:minimumAutoupdateVersion>
</item>
```

If this value is set, it indicates the lowest version that can automatically update to the version referenced by the appcast (i.e. without showing the _update available_ GUI). Apps with a lower `CFBundleVersion` will always see the _update available_ GUI, regardless of their `SUAutomaticallyUpdate` user defaults setting.

Additionally in Sparkle 2:
* A developer can publish a new minor patch release preceding the major release and Sparkle will prefer to install the latest minor release available. For example, if 2.0 is marked as a major release (with `sparkle:minimumAutoupdateVersion` set to 2.0), but 1.9.4 is available then 1.9.4 will be offered first.
* A user can choose to skip out of future update alerts to a major upgrade. For example, if 2.0 is a major release offered and the user is on 1.9.4, they can choose to skip 2.0. Sparkle will confirm with the user if they want to skip automatic alerts for 2.0 and future updates beyond it (eg 2.1). The user can later undo this if they check for updates manually.

## Downloading from a web site

If you want to provide a download link, instead of having Sparkle download and install the update itself, you can omit the `<enclosure>` tag and add `<sparkle:version>` and `<link>` tags. For example:

```xml
<item>
    <title>Version 1.2.4</title>
    <sparkle:version>1.2.4</sparkle:version>
    <sparkle:releaseNotesLink>https://example.com/release_notes_test.html</sparkle:releaseNotesLink>
    <pubDate>Mon, 28 Jan 2013 14:30:00 +0500</pubDate>
    <link>https://myproductwebsite.com</link>
</item>
```

Apps that use Sparkle 2 can use the new `<sparkle:informationalUpdate>` tag instead of omitting the `<enclosure>` tag. This also allows specifying which application versions should see an update as only informational:

```xml
<item>
    <title>Version 1.2.4</title>
    <sparkle:version>1.2.4</sparkle:version>
    <sparkle:informationalUpdate>
    <sparkle:version>1.2.3</sparkle:version>
    </sparkle:informationalUpdate>
    <sparkle:releaseNotesLink>https://example.com/release_notes_test.html</sparkle:releaseNotesLink>
    <pubDate>Mon, 28 Jan 2013 14:30:00 +0500</pubDate>
    <link>https://myproductwebsite.com</link>
    <enclosure url="https://example.com/downloads/app.zip.or.dmg.or.tar.etc"
                sparkle:edSignature="7cLALFUHSwvEJWSkV8aMreoBe4fhRa4FncC5NoThKxwThL6FDR7hTiPJh1fo2uagnPogisnQsgFgq6mGkt2RBw=="
                length="1623481"
                type="application/octet-stream" />
</item>
```

In this example, because `<sparkle:version>1.2.3</sparkle:version>` is specified in `<sparkle:informationalUpdate>`, only version `1.2.3` will see this update as an informational one with a download link. Other versions of the application will see this as an update they can install from inside the application. You can add more children to specify, for example, that version 1.2.2 should also see the update as informational. If you do not specify any children to `<sparkle:informationalUpdate>`, then the update is informational to all versions.

## Critical updates

Updates that are marked critical are shown to the user more promptly and do not let the user skip them. You can use the `<sparkle:criticalUpdate>` tag. For example, if version 1.2.4 is a critical update:

```xml
<item>
    <title>Version 1.2.4</title>
    <sparkle:version>1.2.4</sparkle:version>
    <sparkle:releaseNotesLink>https://example.com/release_notes_test.html</sparkle:releaseNotesLink>
    <pubDate>Mon, 28 Jan 2013 14:30:00 +0500</pubDate>
    <link>https://myproductwebsite.com</link>
    <sparkle:tags>
    <sparkle:criticalUpdate></sparkle:criticalUpdate>
    </sparkle:tags>
    <enclosure url="https://example.com/downloads/app.zip.or.dmg.or.tar.etc"
                sparkle:edSignature="7cLALFUHSwvEJWSkV8aMreoBe4fhRa4FncC5NoThKxwThL6FDR7hTiPJh1fo2uagnPogisnQsgFgq6mGkt2RBw=="
                length="1623481"
                type="application/octet-stream" />
</item>
```

Apps that use Sparkle 2 can use the newer `<sparkle:criticalUpdate>` tag that is a top-level element and not placed within `<sparkle:tags>`. Additionally, the version that was last critical can be specified. For example, when 1.2.5 is released you can specify that only versions less than 1.2.4 should treat this as a critical update:

```xml
<item>
    <title>Version 1.2.5</title>
    <sparkle:version>1.2.5</sparkle:version>
    <sparkle:releaseNotesLink>https://example.com/release_notes_test.html</sparkle:releaseNotesLink>
    <pubDate>Mon, 28 Jan 2013 14:30:00 +0500</pubDate>
    <link>https://myproductwebsite.com</link>
    <sparkle:criticalUpdate sparkle:version="1.2.4"></sparkle:criticalUpdate>
    <enclosure url="https://example.com/downloads/app.zip.or.dmg.or.tar.etc"
                sparkle:edSignature="7cLALFUHSwvEJWSkV8aMreoBe4fhRa4FncC5NoThKxwThL6FDR7hTiPJh1fo2uagnPogisnQsgFgq6mGkt2RBw=="
                length="1623481"
                type="application/octet-stream" />
</item>
```

## Phased group rollouts

Phased group rollouts allows distributing your update to users in different groups and phases. For example on the first day an update is posted, one group of your users may be notified of a new update, but the second group will only be notified of the update on the second day. And so on. This allows posting updates out in the field more gradually.

This feature requires:
* Adding a `<pubDate>` tag in your update item. The `E, dd MMM yyyy HH:mm:ss Z` date format will work.
* Adding a `<sparkle:phasedRolloutInterval>` tag with the specified phased rollout interval between groups in seconds.

For example:

```xml
<item>
    <title>Version 1.2.5</title>
    <sparkle:version>1.2.5</sparkle:version>
    <sparkle:releaseNotesLink>https://example.com/release_notes_test.html</sparkle:releaseNotesLink>
    <pubDate>Mon, 28 Jan 2013 14:30:00 +0500</pubDate>
    <link>https://myproductwebsite.com</link>
    <sparkle:phasedRolloutInterval>86400</sparkle:phasedRolloutInterval>
    <enclosure url="https://example.com/downloads/app.zip.or.dmg.or.tar.etc"
                sparkle:edSignature="7cLALFUHSwvEJWSkV8aMreoBe4fhRa4FncC5NoThKxwThL6FDR7hTiPJh1fo2uagnPogisnQsgFgq6mGkt2RBw=="
                length="1623481"
                type="application/octet-stream" />
</item>
```

This update item specifies the `phasedRolloutInterval` as 86400 seconds, or 1 day. This means that there is 1 day of interval between groups. Sparkle hardcodes the number of groups to 7, so this means that the update will be rolled out completely in 7 days and an update will be rolled out to a new group each day after the `pubDate`.

Sparkle generates a random group ID in the application's user defaults using the `SUUpdateGroupIdentifier` key. This ID is sometimes re-generated (upon downloading an update) and is not transmitted to servers.

Phased group rollouts do not take effect for updates that are marked critical or for when the user manually checks for new updates.

## Channels

Sparkle 2 provides specifying what channel an update is on. Examples of channels may include beta updates or updates that are staged and not ready for production. By default, updaters only look for updates that are on the default channel.

Updates are posted on the default channel unless specified otherwise. This example shows an update posted on the "beta" channel:

```xml
<item>
    <title>Version 2.0 (Beta 1)</title>
    <sparkle:channel>beta</sparkle:channel>
    <sparkle:version>20001</sparkle:version>
    <sparkle:shortVersionString>2.0b1</sparkle:shortVersionString>
</item>
```

Only updaters that are allowed to look in the beta channel can find this update. An updater may use [-[SPUUpdaterDelegate allowedChannelsForUpdater:]](/documentation/api-reference/Protocols/SPUUpdaterDelegate.html#/c:objc(pl)SPUUpdaterDelegate(im)allowedChannelsForUpdater:) to find this update in addition to updates that are on the default channel:

```swift
func allowedChannels(for updater: SPUUpdater) -> Set<String> {
    return Set(["beta"])
}
```

Note that an updater cannot exclude itself from the default channel. Channels are intended to be a way to branch off updates to your application until newer updates are ready to come back to the default channel.

Channels can only be used when all of your users downloading your appcast are running a version of Sparkle that supports them. Sparkle 2 added them as of [June 27, 2021](https://github.com/sparkle-project/Sparkle/pull/1879). You can expedite this process by [switching to a new appcast](#upgrading-to-newer-features). Otherwise older versions of Sparkle may need to set the feed url programmatically below.


## Setting the feed programmatically

The appcast feed URL can be changed programmatically at runtime. If the feed URL is always static and the same, please set it in the Info.plist with the `SUFeedURL` key instead. Even if you have a secondary appcast feed, we recommend keeping a default one specified in the application's Info.plist.

If you want to set the feed programmatically to provide beta/nightly updates, please try to adopt [channels](#channels) in the future instead. If you are supporting older Sparkle versions, continue reading on.

The recommended way to change the feed URL programmatically is using `-[SUUpdaterDelegate feedURLStringForUpdater:]` or [-[SPUUpdaterDelegate feedURLStringForUpdater:]](/documentation/api-reference/Protocols/SPUUpdaterDelegate.html#/c:objc(pl)SPUUpdaterDelegate(im)feedURLStringForUpdater:) in Sparkle 2.

Here is an example:

```swift
func feedURLString(for updater: SPUUpdater) -> String? {
    return UserDefaults.standard.bool(forKey: "beta") ? BETA_FEED_URL_STRING : STABLE_FEED_URL_STRING
}
```

Sparkle also has `-setFeedURL:` or `feedURL` property on the updater which we do not recommend using due to race conditions and user defaults permanence. If you wish to migrate away from this API, you should set the feed URL to nil using this API to clear out any custom feed you have previously set in the bundle's user defaults, so Sparkle doesn't automatically read it.

Web servers can also potentially customize the feed sent back to the client on a per request basis. The [userAgentString](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(py)userAgentString), [httpHeaders](/documentation/api-reference/Classes/SPUUpdater.html#/c:objc(cs)SPUUpdater(py)httpHeaders), and [-[SPUUpdaterDelegate feedParametersForUpdater:sendingSystemProfile:]](/documentation/api-reference/Protocols/SPUUpdaterDelegate.html#/c:objc(pl)SPUUpdaterDelegate(im)feedParametersForUpdater:sendingSystemProfile:) methods from the client may be useful for this.

## Upgrading to newer features

To make use of Sparkle's newest features (such as channels or dropping DSA signatures), you may not be able to adopt these features unless all your users are using a recent version of Sparkle.

One approach is just waiting until the majority of your users are running a recent version of Sparkle and your application.

A second approach is migrating to a new `SUFeedURL` in your new application's `Info.plist`. This ensures that the application versions using your newer appcast have access to newer features.

## Embedded release notes

Instead of linking external release notes using the `<sparkle:releaseNotesLink>` element, you can also embed the release notes directly in the appcast item, inside a `<description>` element. If you wrap it in `<![CDATA[ ... ]]>`, you can use unescaped HTML.

```xml
<item>
    <title>Version 2.0 (2 bugs fixed; 3 new features)</title>
    <link>https://myproductwebsite.com</link>
    <sparkle:version>2.0</sparkle:version>
    <description><![CDATA[
        <h2>New Features</h2>
        ...
    ]]>
    </description>
    ...
</item>
```

You can embed just marked up text (it'll be displayed using standard system font), or a full document with `<!DOCTYPE html><style>`, etc.

## Full release notes

In Sparkle 2, the `<sparkle:fullReleaseNotesLink>` element may be used to specify the full release notes, or version history, link to your product. When the user checks for updates and no new updates are available, Sparkle may let the user open this link in their web browser. If this element is not specified, Sparkle will default to using the `<sparkle:releaseNotesLink>` element instead if present, which may be version specific. Full release notes can also be used if your application uses embedded release notes.

```xml
<item>
    <title>Version 2.0 (2 bugs fixed; 3 new features)</title>
    <link>https://myproductwebsite.com</link>
    <sparkle:version>2.0</sparkle:version>
    <sparkle:releaseNotesLink>https://myproductwebsite.com/app/2.0.html</sparkle:releaseNotesLink>
    <sparkle:fullReleaseNotesLink>https://myproductwebsite.com/app/full-history/</sparkle:fullReleaseNotesLink>
</item>
```

Alternatively, an application that uses Sparkle's standard user interface may implement [-[SPUStandardUserDriverDelegate standardUserDriverShowVersionHistoryForAppcastItem:]](/documentation/api-reference/Protocols/SPUStandardUserDriverDelegate.html#/c:objc(pl)SPUStandardUserDriverDelegate(im)standardUserDriverShowVersionHistoryForAppcastItem:) to show full offline or in-app release notes to the user.

## Localization

You can provide additional release notes for localization purposes. For instance:

```xml
<sparkle:releaseNotesLink>https://example.com/app/2.0.html</sparkle:releaseNotesLink>
<sparkle:releaseNotesLink xml:lang="de">https://example.com/app/2.0_German.html</sparkle:releaseNotesLink>
```

Use the `xml:lang` attribute with the appropriate two-letter country code for each localization. You can also use this attribute with the `<description>` tag.

## Extending the appcast

You can define your own top level elements in the appcast item using your own custom XML namespace (specified by `xmlns`). Here is an example defining `xmlns:myapp` namespace and creating a `myapp:messageOfTheDay` element:

```xml
<?xml version="1.0" encoding="utf-8"?>
<rss version="2.0" xmlns:sparkle="http://www.andymatuschak.org/xml-namespaces/sparkle" xmlns:myapp="https://myproductpage.com/" xmlns:dc="http://purl.org/dc/elements/1.1/">
    <channel>
    <title>My App Changelog</title>
    <language>en</language>
        <item>
        <title>Version 2.0</title>
        <myapp:messageOfTheDay>Tip: you can do X by using Y</myapp:messageOfTheDay>
        <!-- The rest of the feed item -->
        </item>
    </channel>
</rss>
```

In Sparkle 2, one place to parse this custom property is in [-[SPUUpdaterDelegate updaterDidNotFindUpdate:error:]](/documentation/api-reference/Protocols/SPUUpdaterDelegate.html#/c:objc(pl)SPUUpdaterDelegate(im)updaterDidNotFindUpdate:error:). Here's an example:

```swift
func updaterDidNotFindUpdate(_ updater: SPUUpdater, error: Error) {
    let userInfo = (error as NSError).userInfo
    guard
        let latestUpdateItem = userInfo[SPULatestAppcastItemFoundKey] as? SUAppcastItem,
        let userInitiated = userInfo[SPUNoUpdateFoundUserInitiatedKey] as? Bool,
        let reasonValue = userInfo[SPUNoUpdateFoundReasonKey] as? OSStatus,
        let reason = SPUNoUpdateFoundReason(rawValue: reasonValue),
        let messageOfTheDay = latestUpdateItem.propertiesDictionary["myapp:messageOfTheDay"] as? String else {
        return
    }
    
    if !userInitiated &&
        !latestUpdateItem.isMajorUpgrade &&
        !latestUpdateItem.isCriticalUpdate &&
        (reason == .onLatestVersion || reason == .onNewerThanLatestVersion) {
        print(messageOfTheDay)
    }
}
```

## Alternate download locations for other operating systems

Sparkle is available for [Windows](http://winsparkle.org).

To keep the appcast file compatible with the standard Sparkle implementation, a new tag has to be used for cross platform support. It is suggested to use the following to specify downloads for non macOS systems:

```xml
<sparkle:enclosure sparkle:os="os_name" ... />
```

Replace _os_name_ with either "windows" or "linux", respectively (mind the lower case!). Feel free to add other OS names as needed.

## API Specification

A more formal specification of Sparkle's appcast item properties can be found in the [SUAppcastItem API Reference](/documentation/api-reference/Classes/SUAppcastItem.html).