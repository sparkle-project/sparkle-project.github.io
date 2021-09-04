---
layout: documentation
id: documentation
title: Bundles
---

### Sparkle 1

#### Bundles

If you want to support updating for a non-app bundle, such as a Preference Pane, you cannot simply instantiate an SUUpdater instance in your MainMenu.nib as described in [Basic Setup](/documentation/#basic-setup). The reason is that SUUpdater creates a separate shared instance for every bundle, and the instance that is instantiated in a .nib file will always be the instance for the hosting .app bundle. This is the SUUpdater instance returned by `[SUUpdater sharedUpdater]` or `[[SUUpdater alloc] init]`. Note that the latter is used to instantiate an instance of a custom class in a .nib.

For a non-app bundle, you can get the updater instance for your bundle as follows:
`[SUUpdater updaterForBundle:[NSBundle bundleForClass:[self class]]]`
This will return the SUUpdater instance for the bundle in which the class of the calling object is defined.

To use Sparkle for updating a non-app bundle, most of the steps described in [Basic Setup](/documentation/#basic-setup) still apply. Only step 2 will not work. Instead, you should instantiate the SUUpdater instance for your bundle in code at an early stage, just after your bundle was loaded.

For Sparkle version 1.5b6 there is one more consideration. If SUUpdater is instantiated during the launch of the hosting application, before the hosting application has finished launching, there is nothing extra you need to do. However, if your bundle may be loaded after the hosting application finished launching and you want Sparkle to automatically check for updates, you will have to manually start the update cycle. This can be done by simply calling `-resetUpdateCycle` on your updater instance; make sure you set `SUEnableAutomaticChecks` to `YES` in Info.plist to schedule the checks appropriately.

Due to poor design, if you want to use profiling with a bundle, you'll have to proxy a call to applicationDidFinishLaunching to get the permission dialog to appear: <//answers.edge.launchpad.net/sparkle/+question/88790>

#### Subclassing SUUpdater

An alternative approach is to use a subclass of SUUpdater whose shared instance is the updater for your bundle. You can then also instantiate this class in a .nib, so you can essentially follow step 2 in [Basic Setup](/documentation/#basic-setup) to initialize your updater. Make sure you also read in the header for your SUUpdater subclass in your .nib.

The subclass only needs to implement the following two methods:

```objc
+ (id)sharedUpdater
{
    return [self updaterForBundle:[NSBundle bundleForClass:[self class]]];
}

- (id)init
{
    return [self initForBundle:[NSBundle bundleForClass:[self class]]];
}
```

### Sparkle 2

#### Bundles

The story with updating bundles between Sparkle 1 and 2 is a bit different.

First, Sparkle 2 supports updating any Sparkle-based bundle, which is not just limited to your own application or process. The updater distinguishes the `hostBundle` from the `applicationBundle`. The `hostBundle` is the bundle that Sparkle updates and the `applicationBundle` is the bundle that can be terminated and re-launched (if applicable). See the [SPUUpdater API Reference](/documentation/api-reference/Classes/SPUUpdater.html) for more information on instantiating your own updater. Some kinds of plug-ins may also want to use `SURelaunchHostBundle` key in [Customizing Sparkle](/documentation/customization) for re-launching the host bundle.

Second, Sparkle 2 doesn't keep track of shared updater instances and doesn't try to prohibit multiple updater instances from existing  -- either inside the same process or multiple updaters updating the same bundle from different processes. In fact, it is even possible for one updater to start an update (say a silent deferred one), and for a second updater (say sparkle-cli) to resume that same update for the same bundle.

If you want to update a plug-in, injecting Sparkle.framework or sharing a version of Sparkle with the host process is not advisable. Instead, look into calling out to an out-of-process tool that can update the plug-in like [sparkle-cli](/documentation/sparkle-cli) can.

If your plug-in inherits a sandbox environment, you may need to use a companion application with XPC Services and follow Sparkle's [Sandboxing guide](/documentation/sandboxing).
