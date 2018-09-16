---
layout: documentation
id: documentation
title: Delta Updates
---
## Delta Updates

Sparkle supports "delta updates" for your application: when possible, users can download only the bits that have changed. These updates are much smaller and faster than DMG.

For each new version you release, you can provide a list of `.delta` files in addition to the "full" archive of the version. Each `.delta` contains the information necessary to upgrade from a single older version.

If the user is running a version of the app for which you haven't provided a `.delta`, or if the patch doesn't apply cleanly, they'll use the non-delta "full" update.

### Automated generation

The `./bin/generate_appcast` tool that comes with Sparkle [automatically generates and signs delta updates](/documentation/#publish-your-appcast).

### Manual generation

To generate a `.delta`, you use the `BinaryDelta` tool included with Sparkle like this:

    BinaryDelta create path/to/old/MyApp.app path/to/new/MyApp.app patch.delta

To verify that your generated patch applies correctly:

    BinaryDelta apply path/to/old/MyApp.app path/to/patched/MyApp.app patch.delta

Version 2 patches created by BinaryDelta are not backwards compatible with Sparkle 1.9 or earlier. Pass `--version=1` when creating patches only if you want old versions to apply them.

<div class="alert alert-warning" role="alert">
<strong>Note:</strong> We don't recommend creating delta patches from an application that uses Sparkle 1.10, due to a potential crash in the updater for applying the updates.
</div>

[Sign each `.delta` file](/documentation/#segue-for-security-concerns) and add an enclosure to your appcast's `<item>` in `<sparkle:deltas>` for each `.delta` file:

    <item>
       <title>Version 2.0 </title>
       <description>foo bar baz</description>
       <pubDate>Wed, 09 Jan 2006 19:20:11 +0000</pubDate>
       <enclosure url="http://you.com/2.0_full.zip"
                  sparkle:version="2.0"
                  length="123"
                  type="application/octet-stream"
                  sparkle:edSignature="..." />
       <sparkle:deltas>
           <enclosure url="http://you.com/1.5-2.0.delta"
                      sparkle:version="2.0"
                      sparkle:deltaFrom="1.5"
                      length="1485"
                      type="application/octet-stream"
                      sparkle:edSignature="..." />
           <enclosure url="http://you.com/1.6-2.0.delta"
                      sparkle:version="2.0"
                      sparkle:deltaFrom="1.6"
                      length="1428"
                      type="application/octet-stream"
                      sparkle:edSignature="..." />
       </sparkle:deltas>
    </item>

### Tips for Improving Download Size & Performance

We recommend reading Apple's article on [reducing the download size for iOS app updates](https://developer.apple.com/library/content/qa/qa1779/_index.html), which is applicable here too. To reword:

* Do not make unnecessary modifications to files. Compare the contents of the prior and new versions of your app and verify that you've only changed what you expect within your app bundle. Running `BinaryDelta --verbose` will show a diff of what has changed.
* Content that you expect to change in an update should be stored in separate files from content that you don't expect to change. This reduces the size of the delta update and increases its install speed.
* Avoid renaming files or folders.

