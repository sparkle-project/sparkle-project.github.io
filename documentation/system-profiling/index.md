---
layout: documentation
id: documentation
title: System Profiling
---
## System Profiling

Sparkle supports optionally sending anonymous information about the user's system when checking for updates. Developers can use this information to better decide what platforms and hardware to support.

### Collected data

* macOS version
* CPU type/subtype (see mach/machine.h for decoder information on this data)
* Mac model
* Number of CPUs/CPU cores,
* 32-bit vs. 64-bit
* CPU speed
* RAM size
* Application name (as indicated by CFBundleName)
* Application version (as indicated by CFBundleVersion)
* User's preferred language
* Anything else you'd like to send along (via a delegate method; see [Customizing Sparkle](/documentation/customization/))

This information will be provided via GET, so if your feed URL is `https://example.org/app.xml`, the request will be made to `https://example.org/app.xml?cpusubtype=4&ncpu=2&appName=App.app&cpuFreqMHz=1830[...]`.

**Note:** In order to standardize the statistics across a userbase with varying update check intervals, Sparkle submits profiling information only once per week.

### Making use of system profiling information

First, you've got to tell Sparkle you want this data to be sent. You can do this by adding a `SUEnableSystemProfiling` key to your app's Info.plist and setting its value to YES.

Next, you've got to put something up server-side to collate and display the submitted information in a useful way (see the list below).

Finally, set the `SUFeedURL` to your server-side script (eg. `https://example.org/profileInfo.php`) instead of the appcast. The script in turn should be set to link to the appcast.

### Known Backends

* Tom Harrington's original PHP-based backend: [download it here](/files/php_sparkle_stats_server.zip)
* Full-featured backend for CakePHP: [on GitHub here](//github.com/balthisar/JDSparkle)
* Ruby on Rails app for collecting statistics and displaying graphs: [Sparkler](//github.com/mackuba/sparkler)
* Playful weekly user-profile visualizer in PHP. No DB required: [Sparkle-Posse](//habilis.net/sparkle-posse/)
