---
layout: documentation
id: documentation
title: CocoaPods
---
## CocoaPods

Installing with [CocoaPods](//cocoapods.org/pods/Sparkle) is identical to the [normal install](/documentation/#basic-setup) except for step 1.

### 1. Add the Sparkle framework to your project

* Add `pod 'Sparkle'` to your Podfile.
* The Sparkle framework must be signed. CocoaPods doesn't seem to handle this automatically, so a build script is required (courtesy of [furbo.org](http://furbo.org/2013/10/17/code-signing-and-mavericks/)).
* Add a 'Run Script' build phase after the 'Copy Pods Resources' phase and enter the following script (you'll need to substitute 'Example' with your own iTunes identity):

      LOCATION="${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}"

      # Usually set by Xcode
      CODE_SIGN_IDENTITY="Developer ID Application: Example"

      codesign --verbose --force --sign "$CODE_SIGN_IDENTITY" "$LOCATION/Sparkle.framework/Versions/A"

That's all. Build your project to confirm that it builds and signs the framework, then continue with the [basic setup](/documentation/#set-up-a-sparkle-updater-object).

The Sparkle Project maintains its own CocoaPod Podspec and it is kept up-to-update in sync with official releases.
