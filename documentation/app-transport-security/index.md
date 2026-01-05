---
layout: documentation
id: documentation
title: Sparkle App Transport Security
---
## App Transport Security

Apple has deprecated insecure HTTP. Applications are by default **blocked from using HTTP** and will not be able to download any updates over HTTP. You **must** either:

* Switch to HTTPS entirely. Make the appcast URL in Info.plist and download URLs in the appcast use HTTPS. This is recommended, and you can get free certificates from [Let's Encrypt](//letsencrypt.org/) or [AWS Certificate Manager](//aws.amazon.com/certificate-manager/).

* Add an exception to Info.plist in your app if you can't stop using HTTP yet. [The exception properties are in Apple's documentation](//developer.apple.com/library/prerelease/mac/technotes/App-Transport-Security-Technote/).

App Transport Security has other specific requirements too, so please test updating your app even if you already are serving over HTTPS!

[‹ Back to documentation](/documentation/#segue-for-security-concerns)
