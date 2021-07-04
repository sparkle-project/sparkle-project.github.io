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

    ditto -c -k --sequesterRsrc --keepParent <src_path_to_app> <zip_dest>

If you can't use regular app bundles, you can also create an Installer .pkg with the same name as your app and put that .pkg in one of the aforementioned archive formats. By default Sparkle launches Installer without a GUI. If instead of .pkg extension you use .sparkle_interactive.pkg, then installation will run with a GUI and ask user to confirm every step.

### Secure your update

In order to prevent corruption and man-in-the-middle attacks against your users, you must cryptographically sign your updates.

Signatures are automatically generated when you make an appcast using `generate_appcast` tool. This is a recommended method.

To manually generate signatures for your updates, Sparkle includes a tool to help you make a EdDSA signature of the archive. From the Sparkle distribution:

    ./bin/sign_update path_to_your_update.zip

The output will be an XML fragment with your update's EdDSA signature and (optional) file size; you'll add this attribute to your enclosure in the next step.

Since 10.11, macOS has [App Transport Security](//developer.apple.com/library/prerelease/mac/technotes/App-Transport-Security-Technote/) policy which blocks apps from using insecure HTTP connections. This restriction applies to Sparkle as well, so you will need to serve your appcast and the update files over HTTPS.

### Update your appcast

You need to create an `<item>` for your update in your appcast. See the [sample appcast](/files/sparkletestcast.xml) for an example. Here's a template you might use:

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

Test your update, and you're done!

Note on `sparkle:version`: Our previous documentation used to recommend specifying `sparkle:version` (and `sparkle:shortVersionString`) as part of the `enclosure` item. While this works fine, for overall consistency we now recommend specifying them as top level items instead as shown here. This is supported across all versions of Sparkle.

## Delta updates

If your app is large, or if you're updating primarily only a small part of it, you may find [delta updates](/documentation/delta-updates/) useful: they allow your users to only download the bits that have changed. The `generate_appcast` tool automatically generates delta updates.

## Internal build numbers

If you use internal build numbers for your `CFBundleVersion` key and a human-readable `CFBundleShortVersionString`, you can make Sparkle hide the internal version from your users.

Set the `sparkle:version` element in your item to the internal, machine-readable version (ie: "1248"). Then set a `sparkle:shortVersionString` element on in your item to the human-readable version (ie: "1.5.1"). For example:

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

Note that the internal version number (`CFBundleVersion` and `sparkle:version`) is intended to be machine-readable and is not generally suitable for formatted text or git changeset IDs.

## Minimum system version requirements

If an update to your application raises the required version of macOS, you can only offer that update to qualified users.

Add a `sparkle:minimumSystemVersion` child to the `<item>` in question specifying the required system version, such as "10.11.0" (be sure to specify a three-part version in form of *major.minor.patch*):

    <item>
        <title>Version 2.0 (2 bugs fixed; 3 new features)</title>
        <link>https://myproductwebsite.com</link>
        <sparkle:version>2.0</sparkle:version>
        <sparkle:minimumSystemVersion>10.11.0</sparkle:minimumSystemVersion>
    </item>

Note that Sparkle only works with macOS 10.9 or later (macOS 10.11 or later for Sparkle 2), so that's the lowest minimum version you can use.

Sparkle also supports a `sparkle:maximumSystemVersion` element that can limit the maximum system version similarly.

Additionally in Sparkle 2, if the user checks for new updates manually and the cannot update because of an operating system requirement, the standard updater alert will inform the user their operating system is incompatible and provide them an option to visit your website specified by the `<link>` element.

## Major upgrades

If an update to your application is a major or paid upgrade, you may want to prevent the update from being installed automatically.

Add a `sparkle:minimumAutoupdateVersion` child to the `<item>` in question specifying the major update's `<CFBundleVersion>`, such as "2.0":

    <item>
        <title>Version 2.1 (2 bugs fixed; 3 new features)</title>
        <link>https://myproductwebsite.com</link>
        <sparkle:version>2.1</sparkle:version>
        <sparkle:minimumAutoupdateVersion>2.0</sparkle:minimumAutoupdateVersion>
    </item>

If this value is set, it indicates the lowest version that can automatically update to the version referenced by the appcast (i.e. without showing the _update available_ GUI). Apps with a lower `CFBundleVersion` will always see the _update available_ GUI, regardless of their `SUAutomaticallyUpdate` user defaults setting.

Additionally in Sparkle 2:
* A developer can publish a new minor patch release preceding the major release and Sparkle will prefer to install the latest minor release available. For example, if 2.0 is marked as a major release (with `sparkle:minimumAutoupdateVersion` set to 2.0), but 1.9.4 is available then 1.9.4 will be offered first.
* A user can choose to skip out of future update alerts to a major upgrade. For example, if 2.0 is a major release offered and the user is on 1.9.4, they can choose to skip 2.0. Sparkle will confirm with the user if they want to skip automatic alerts for 2.0 and future updates beyond it (eg 2.1). The user can later undo this if they check for updates manually.

## Downloading from a web site

If you want to provide a download link, instead of having Sparkle download and install the update itself, you can omit the `<enclosure>` tag and add `<sparkle:version>` and `<link>` tags. For example:

    <item>
      <title>Version 1.2.4</title>
      <sparkle:version>1.2.4</sparkle:version>
      <sparkle:releaseNotesLink>https://example.com/release_notes_test.html</sparkle:releaseNotesLink>
      <pubDate>Mon, 28 Jan 2013 14:30:00 +0500</pubDate>
      <link>https://myproductwebsite.com</link>
    </item>

Apps that use Sparkle 2 can use the new `<sparkle:informationalUpdate>` tag instead of omitting the `<enclosure>` tag. This also allows specifying which application versions should see an update as only informational:

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

In this example, because `<sparkle:version>1.2.3</sparkle:version>` is specified in `<sparkle:informationalUpdate>`, only version `1.2.3` will see this update as an informational one with a download link. Other versions of the application will see this as an update they can install from inside the application. You can add more children to specify, for example, that version 1.2.2 should also see the update as informational. If you do not specify any children to `<sparkle:informationalUpdate>`, then the update is informational to all versions.

## Critical updates

Updates that are marked critical are shown to the user more promptly and do not let the user skip them. You can use the `<sparkle:criticalUpdate>` tag. For example, if version 1.2.4 is a critical update:

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

Apps that use Sparkle 2 can use the newer `<sparkle:criticalUpdate>` tag that is a top-level element and not placed within `<sparkle:tags>`. Additionally, the version that was last critical can be specified. For example, when 1.2.5 is released you can specify that only versions less than 1.2.4 should treat this as a critical update:

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

## Phased Group Rollouts

Phased group rollouts allows distributing your update to users in different groups and phases. For example on the first day an update is posted, one group of your users may be notified of a new update, but the second group will only be notified of the update on the second day. And so on. This allows posting updates out in the field more gradually.

This feature requires:
* Adding a `<pubDate>` tag in your update item. The `E, dd MMM yyyy HH:mm:ss Z` date format will work.
* Adding a `<sparkle:phasedRolloutInterval>` tag with the specified phased rollout interval between groups in seconds.

For example:

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

This update item specifies the `phasedRolloutInterval` as 86400 seconds, or 1 day. This means that there is 1 day of interval between groups. Sparkle hardcodes the number of groups to 7, so this means that the update will be rolled out completely in 7 days and an update will be rolled out to a new group each day after the `pubDate`.

Sparkle generates a random group ID in the application's user defaults using the `SUUpdateGroupIdentifier` key. This ID is sometimes re-generated (upon downloading an update) and is not transmitted to servers.

Phased group rollouts do not take effect for updates that are marked critical or for when the user manually checks for new updates.

## Embedded release notes

Instead of linking external release notes using the `<sparkle:releaseNotesLink>` element, you can also embed the release notes directly in the appcast item, inside a `<description>` element. If you wrap it in `<![CDATA[ ... ]]>`, you can use unescaped HTML.

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

You can embed just marked up text (it'll be displayed using standard system font), or a full document with `<!DOCTYPE html><style>`, etc.

## Localization

You can provide additional release notes for localization purposes. For instance:

    <sparkle:releaseNotesLink>https://example.com/app/2.0.html</sparkle:releaseNotesLink>
    <sparkle:releaseNotesLink xml:lang="de">https://example.com/app/2.0_German.html</sparkle:releaseNotesLink>

Use the `xml:lang` attribute with the appropriate two-letter country code for each localization. You can also use this attribute with the `<description>` tag.

## Alternate download locations for other operating systems

Sparkle is available for [Windows](http://winsparkle.org).

To keep the appcast file compatible with the standard Sparkle implementation, a new tag has to be used for cross platform support. It is suggested to use the following to specify downloads for non macOS systems:

    <sparkle:enclosure sparkle:os="os_name" ...>

Replace _os_name_ with either "windows" or "linux", respectively (mind the lower case!). Feel free to add other OS names as needed.
