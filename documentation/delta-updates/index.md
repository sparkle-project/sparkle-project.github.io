---
layout: documentation
id: documentation
title: Delta Updates
---
## Delta Updates

Sparkle supports "delta updates" for your application: when possible, users can download only the bits that have changed. These updates can be much smaller to download and faster to install.

For each new version you release, you can provide a list of `.delta` files in addition to the "full" archive of the version. Each `.delta` contains the information necessary to upgrade from a single older version.

If the user is running a version of the app for which you haven't provided a `.delta`, or if the patch doesn't apply cleanly, they'll use the non-delta "full" update.

<div class="alert alert-warning" role="alert">
<strong>Note:</strong> Due to a macOS change, new delta patches must be created using generate_appcast or BinaryDelta from a Sparkle 1.27.0 or later distribution. Note that newly generated patches are still backwards compatible and can be patched from applications using versions of Sparkle framework older than 1.27.0.
</div>

### Automated generation

The `./bin/generate_appcast` tool that comes with Sparkle [automatically generates and signs delta updates](/documentation/#5-publish-your-appcast).

### Manual generation

To generate a `.delta`, you use the `BinaryDelta` tool included with Sparkle like this:

```sh
BinaryDelta create --version=$FORMAT_VERSION path/to/old/MyApp.app path/to/new/MyApp.app patch.delta
```

Please see the [compatibility section](#compatibility) below on what `$FORMAT_VERSION` to use.

To verify that your generated patch applies correctly:

```sh
BinaryDelta apply path/to/old/MyApp.app path/to/patched/MyApp.app patch.delta
```

[Sign each `.delta` file](/documentation/#3-segue-for-security-concerns) and add an enclosure to your appcast's `<item>` in `<sparkle:deltas>` for each `.delta` file:

```xml
<item>
   <title>Version 2.0 </title>
   <link>https://myproductwebsite.com</link>
   <sparkle:version>2.0</sparkle:version>
   <description>foo bar baz</description>
   <pubDate>Wed, 09 Jan 2006 19:20:11 +0000</pubDate>
   <enclosure url="http://you.com/2.0_full.zip"
              length="123"
              type="application/octet-stream"
              sparkle:edSignature="..." />
   <sparkle:deltas>
       <enclosure url="http://you.com/1.5-2.0.delta"
                  sparkle:deltaFrom="1.5"
                  length="1485"
                  type="application/octet-stream"
                  sparkle:edSignature="..." />
       <enclosure url="http://you.com/1.6-2.0.delta"
                  sparkle:deltaFrom="1.6"
                  length="1428"
                  type="application/octet-stream"
                  sparkle:edSignature="..."
                  sparkle:deltaFromSparkleExecutableSize="1844096"
                  sparkle:deltaFromSparkleLocales="en,ca,fr,hr,hu" />
   </sparkle:deltas>
</item>
```

Sparkle 2.3 (beta) introduces two optional attributes for testing if a delta update can be applied before downloading a delta update. These are the `sparkle:deltaFromSparkleExecutableSize` and `sparkle:deltaFromSparkleLocales` attributes. The `sparkle:deltaFromSparkleExecutableSize` attribute refers to the file size in bytes of `Sparkle.framework/Versions/B/Sparkle` from the older application. The `deltaFromSparkleLocales` refers to a comma-delimited partial list of `.lproj` locale directory names (minus the path extension) in `Sparkle.framework/Versions/B/Resources/` from the old application. These attributes help Sparkle detect cases where a user strips your application's localizations or architecture slices. In these cases, Sparkle can skip attempting to download and apply a delta update.

### Compatibility

When you create a delta update between two versions of your application, you must ensure you are using a delta format version that your older application knows how to patch. Below is a table describing the requirements for each binary delta version.

| Delta Format   | Supports                      | Changes                                                                                        |
| --------------- | ----------------------------- | ---------------------------------------------------------------------------------------------------- |
| 3               | Sparkle 2.1                   | More efficient custom delta container format, lzma compression + other compression options, file rename heuristic tracking. |
| 2               | Sparkle 1.10                  | Improved and changed hash function for reducing collisions.                                          |
| 1               | Sparkle 1.5                   | Added initial binary delta format using libxar and bzip2 compression. Tracking of insertions, deletions, and binary file diffs using bsdiff. |

Note if you are using `generate_appcast`, picking the delta version to use is automatically handled. If you are using `BinaryDelta create` though, you will need to pass the appropriate delta version via the `--version` argument.

Older delta format versions will eventually be phased out. Please do not create new patches using an older version than necessary. Always use the latest tools when creating delta patches because they may contain minor bug fixes that don't require a major format change.

In any case where a binary delta update fails to install, Sparkle falls back to downloading and installing the regular full update. Sparkle verifies a checksum to ensure the resulting application has been patched successfully.

### Tips for Improving Download Size & Performance

We recommend reading Apple's article on [reducing the download size for iOS app updates](https://developer.apple.com/library/content/qa/qa1779/_index.html), which is applicable here too. To reword:

* Do not make unnecessary modifications to files. Compare the contents of the prior and new versions of your app and verify that you've only changed what you expect within your app bundle. Running `BinaryDelta --verbose` will show a diff of what has changed.
* Content that you expect to change in an update should be stored in separate files from content that you don't expect to change. This reduces the size of the delta update and increases its install speed.
* Avoid renaming files or folders. Our version 3 delta format has heuristics for tracking when files are moved around in some cases however.

### Metadata Specification

Binary delta updates do not support and will reject the following metadata:

* Access control lists (ACLs) in either the old or new application.
* Extended attributes containing code signing information in either the old or new application. This can be resolved by properly structuring your application such that data and code are placed in the correct directories per [Code Signing in Depth guide](https://developer.apple.com/library/archive/technotes/tn2206/_index.html#//apple_ref/doc/uid/DTS40007919-CH1-TNTAG201). Only mach-o binaries should embed code signing information.
* Creating or applying diffs on file systems which do not preserve regular permission modes like FAT are not supported. Sparkle 2.2 and later performs a permission validation test before downloading delta updates. Sparkle's executable must have file permissions `0755`.

Binary delta updates will pass but ignore or warn about the following metadata:

* Extended attributes changes on files are not preserved and are just ignored.
* File permissions on symbolic links that are not `0755` are ignored, and `0755` will always be used instead. A warning will be issued. Some filesystems, e.g. Linux ones, do not support file permissions on symbolic links.
* Irregular file permissions on files that are not `0755` or `0644` will be respected, but a warning may be issued (from Sparkle 2.1 onwards).
* Custom icons users set using a resource fork (e.g. using Finder's `Get Info` window) are ignored/preserved when applying patches, but prohibited when creating patches (from Sparkle 2.2 onwards).

Binary delta updates do support the following meta changes:

* File permission changes.
* File type (regular, symbolic link, directory) changes.
* File case (from `foo` -> `FOO`) changes.
* HFS+ or file system level compression in the version 3 format. If files from the old bundle are detected to be using this type of compression, then we apply HFS+ compression onto the new app in its entirety via `ditto --hfsCompression` (i.e, we don't track this compression on a per file basis).
