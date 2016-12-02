---
layout: documentation
id: documentation
title: CocoaPods
---
## CocoaPods

Installing with [CocoaPods](//cocoapods.org/pods/Sparkle) is identical to the [normal install](/documentation/#basic-setup) except for step 1.

### 1. Add the Sparkle framework to your project

* Add `pod 'Sparkle'` to your Podfile.
* Add or uncomment `use_frameworks!` in your Podfile.

### 2. Code Signing Sparkle

**If you are using Xcode 7 or later, this step is automatically handled and should be skipped.**

If the application is signed, then the framework must be also signed via a build script (courtesy of [furbo.org](http://furbo.org/2013/10/17/code-signing-and-mavericks/)).

Add a 'Run Script' build phase after the 'Copy Pods Resources' phase and enter the following script (you'll need to substitute 'Example' with your own iTunes identity):

      LOCATION="${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}"

      # Usually set by Xcode
      CODE_SIGN_IDENTITY="Developer ID Application: Example"

      codesign --verbose --force --sign "$CODE_SIGN_IDENTITY" "$LOCATION/Sparkle.framework/Versions/A"

### 3. Finalizing
Build your project to confirm that it builds and signs the framework, then continue with the [basic setup](/documentation/#set-up-a-sparkle-updater-object).

The Sparkle Project maintains its own CocoaPod Podspec and it is kept up-to-update in sync with official releases.
