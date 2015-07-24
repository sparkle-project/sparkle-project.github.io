---
layout: documentation
id: documentation
title: Publishing an update
---
## Publishing an update

So you're ready to release a new version of your app. How do you go about doing that?

### Archive your app

Put a copy of your .app (with the same name as the version it's replacing) in a .zip, .tar.gz, or .tar.bz2. If you distribute your .app in a .dmg, do not zip up the .dmg.

Alternatively, create an Installer .pkg with the same name as your app and put that .pkg in one of the aforementioned archive formats.

### Secure your update

In order to prevent man-in-the-middle attacks against your users, you must cryptographically sign your updates. You can do that using Apple's code signing tools, with a Developer ID.

If you can't code sign your app, you can make a DSA signature of the archive instead. Sparkle includes a script to help you sign your update. From the Sparkle distribution:

    ./bin/sign_update.sh path_to_your_update.zip path_to_your_dsa_priv.pem

The output string is your update's DSA signature; you'll add this as an attribute to your enclosure in the next step. You can remove any newlines in this string.

### Update your appcast

You need to create an `<item>` for your update in your appcast. See the [sample appcast](/files/sparkletestcast.xml) for an example. Here's a template you might use:

    <item>
        <title>Version 2.0 (2 bugs fixed; 3 new features)</title>
        <sparkle:releaseNotesLink>
            http://you.com/app/2.0.html
        </sparkle:releaseNotesLink>
        <pubDate>Wed, 09 Jan 2006 19:20:11 +0000</pubDate>
        <enclosure url="http://you.com/app/Your%20Great%20App%202.0.zip"
                   sparkle:version="2.0"
                   sparkle:dsaSignature="MC0CFBfeCa1JyW30nbkBwainOzrN6EQuAh=" <!-- if you're not code signing -->
                   length="1623481"
                   type="application/octet-stream" />
    </item>

Test your update, and you're done!

## Downloading from a web site

If you want to provide a download link, instead of having Sparkle download and install the update itself, you can omit the `<enclosure>` tag and add `<sparkle:version>` and `<link>` tags. For example:

    <item>
      <title>Version 1.2.4</title>
      <sparkle:releaseNotesLink>http://foo.bar/storage/downloads/macosx/release_notes_test.html</sparkle:releaseNotesLink>
      <pubDate>Mon, 28 Jan 2013 14:30:00 +0500</pubDate>
      <sparkle:version>1.2.4</sparkle:version>
      <link>http://foo.bar/download_link.html_or_dmg_whatever</link>
    </item>

## Delta updates

If your app is large, or if you're updating primarily only a small part of it, you may find [delta updates](/documentation/delta-updates) useful: they allow your users to only download the bits that have changed.

## Internal build numbers

If you use internal build numbers for your CFBundleVersion key (like an SVN revision number) and a human-readable CFBundleShortVersionString, you can make Sparkle hide the internal version from your users.

Set the sparkle:version attribute on your enclosure to the internal version (ie: "248"). Then set a sparkle:shortVersionString attribute on the enclosure to the human readable version (ie: "2.0").

[Remember](http://lists.apple.com/archives/carbon-dev/2006/Jun/msg00139.html) that the internal version number (CFBundleVersion and sparkle:version) is intended to be machine-readable and is not generally suitable for formatted text.

## Minimum system version requirements

If an update to your application raises the required version of OS X, you can only offer that update to qualified users.

Add a sparkle:minimumSystemVersion child to the `<item>` in question specifying the required system version, such as "10.8.4":

    <item>
        <title>Version 2.0 (2 bugs fixed; 3 new features)</title>
        <sparkle:minimumSystemVersion>10.8.4</sparkle:minimumSystemVersion>
    </item>

## Embedded release notes

Instead of linking external release notes using the `<sparkle:releaseNotesLink>` element, you can also embed the release notes directly in the appcast item, inside a `<description>` element. If you wrap it in `<![CDATA[ ... ]]>`, you can use unescaped HTML.

    <item>
        <title>Version 2.0 (2 bugs fixed; 3 new features)</title>
        <description><![CDATA[
            <h2>New Features</h2>
            ...
        ]]>
        </description>
        ...
    </item>

## Localization

You can provide additional release notes for localization purposes. For instance:

    <sparkle:releaseNotesLink>http://you.com/app/2.0.html</sparkle:releaseNotesLink>
    <sparkle:releaseNotesLink xml:lang="de">http://you.com/app/2.0_German.html</sparkle:releaseNotesLink>

Use the `xml:lang` attribute with the appropriate two-letter country code for each localization. You can also use this attribute with the `<description>` tag.

## Xcode integration

[Here's a description](https://web.archive.org/web/20120708050000/http://www.entropy.ch/blog/Developer/2008/09/22/Sparkle-Appcast-Automation-in-Xcode.html) of how to automate the archiving and signing process with a script build phase in Xcode.

## Alternate download locations for other operating systems

Sparkle is available for [Windows](http://winsparkle.org).

To keep the appcast file compatible with the standard Sparkle implementation, a new tag has to be used for cross platform support. It is suggested to use the following to specify downloads for non OS X systems:

    <sparkle:enclosure sparkle:os="os_name" ...>

Replace _os_name_ with either "windows" or "linux", respectively (mind the lower case!). Feel free to add other OS names as needed.
