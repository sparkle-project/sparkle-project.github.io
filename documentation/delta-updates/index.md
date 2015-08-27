---
layout: documentation
id: documentation
title: Delta Updates
---
## Delta Updates

Sparkle supports "delta updates" for your application: when possible, users can download only the bits that have changed. Note that you will need to [generate DSA keys](/documentation#segue-for-security-concerns) for your application.

For each new version you release, you can provide a list of `.delta` files in addition to the "full" archive of the version. Each `.delta` contains the information necessary to upgrade from a single older version.

To generate a `.delta`, you use the `BinaryDelta` tool included with Sparkle like this:

    BinaryDelta create path/to/old/MyApp.app path/to/new/MyApp.app patch.delta

To verify that your generated patch applies correctly:

    BinaryDelta apply path/to/old/MyApp.app path/to/patched/MyApp.app patch.delta

Version 2 patches created by BinaryDelta are not backwards compatible with Sparkle 1.9 or earlier. Pass `--version=1` when creating patches only if you want old versions to apply them.

Note we don't recommend creating delta patches from an application that uses Sparkle 1.10, due to a potential crash in the updater for applying the updates.

Presently, you'll have to build Sparkle from source to get the `BinaryDelta` tool. You don't need to distribute the tool with your app.

Then you sign each `.delta` and add an entry to your appcast's `<item>` for each `.delta` you have created:

    <item>
       <title>Version 2.0 </title>
       <description>foo bar baz</description>
       <pubDate>Wed, 09 Jan 2006 19:20:11 +0000</pubDate>
       <enclosure url="http://you.com/2.0_full.zip"
                  sparkle:version="2.0"
                  length="123"
                  type="application/octet-stream"
                  sparkle:dsaSignature="..." />
       <sparkle:deltas>
           <enclosure url="http://you.com/1.5-2.0.delta"
                      sparkle:version="2.0"
                      sparkle:deltaFrom="1.5"
                      length="1485"
                      type="application/octet-stream"
                      sparkle:dsaSignature="..." />
           <enclosure url="http://you.com/1.6-2.0.delta"
                      sparkle:version="2.0"
                      sparkle:deltaFrom="1.6"
                      length="1428"
                      type="application/octet-stream"
                      sparkle:dsaSignature="..." />
       </sparkle:deltas>
    </item>

If the user is running a version of the app for which you haven't provided a `.delta`, or if the patch doesn't apply cleanly, they'll use the non-delta "full" update.
