---
layout: documentation
id: documentation
title: Upgrading to EdDSA from DSA
---

Sparkle before version 1.21 used to use only older DSA signatures, which are now deprecated. They are however still supported for migration purposes. We strongly recommend migrating over to EdDSA (ed25519) and planning to drop DSA signatures in new updates of your applications.

Please read all the sections on this page to get a full picture of how EdDSA migration works.

This guide assumes you are currently shipping updates with a DSA public key using `SUPublicDSAKeyFile`. If you are using `SUPublicDSAKey` key instead, the information below still applies.

## Upgrading from Sparkle 1.27 or later

The most latest versions of Sparkle have complete support in dropping DSA public keys and signatures in new application updates. This includes Sparkle 1.27 and Sparkle 2 (as of [July 10, 2021](https://github.com/sparkle-project/Sparkle/pull/1888)). These versions are also optimized by not verifying DSA signatures if both EdDSA and DSA keys are present.

You may first need to ship an update that provides both DSA and EdDSA public keys (`SUPublicDSAKeyFile` and `SUPublicEDKey`) and signatures (`sparkle:dsaSignature` and `sparkle:edSignature`) however before being able to drop DSA. Application updates signed with a Developer ID certificate may be an exception and be able to shortcut this though.

Read the below sections for more details. Chances are they apply to you as you probably have users running earlier versions of Sparkle.

## Upgrading from Sparkle 1.21 or later for Developer ID signed applications

If all of your users are running Sparkle 1.21 or Sparkle 2 (as of [June 10, 2020](https://github.com/sparkle-project/Sparkle/pull/1553)) which introduced EdDSA support and your old and new updates are signed with the same Apple Developer ID code signing certificate, you are able to just remove the DSA public key (`SUPublicDSAKeyFile`) in your new update and remove the DSA signature (`sparkle:dsaSignature`) in your appcast. Your new update will need a EdDSA public key (`SUPublicEDKey`) and EdDSA signature (`sparkle:edSignature`) in the appcast.

This works because Sparkle uses the matching Apple code signature to trust the change in Sparkle public keys in your new application update. Note to test this, you may need to run a build of your application that uses the same code signing certificate (and not one used only for development).

This approach cannot be used if users are running a version of Sparkle before 1.21 or before the specified Sparkle 2 version above, which do not understand EdDSA keys.

## Other upgrade paths

If one of these applies to you:

* You have users running versions of Sparkle below 1.21 (that do not understand EdDSA)
* You have users running a version of your application that only uses DSA keys, and you do not use Apple code signing or the new and old updates are not signed with the same Deveolper ID certificate.
* You ship package installer upgrades and have users using a version of your application that only uses DSA keys
* You ship package installer upgrades and all your users are using a version of your application that has an EdDSA key, but your users are running a version of Sparkle below 1.27

Then we recommend migrating in two steps.

First, ship a new update using Sparkle 1.27 or later that specifies both DSA and EdDSA public keys (`SUPublicDSAKeyFile` and `SUPublicEDKey`) and signatures in your appcast (`sparkle:dsaSignature` and `sparkle:edSignature`). For example, the appcast item enclosure may look similar to:

```xml
<enclosure url="http://sparkle-project.org/myupdate.zip" sparkle:version="2" type="application/octet-stream" sparkle:dsaSignature="MCwCFAdLjBvvjJN0fnfMnn7BCl650VqNAhR+XuFABVK2y8zJn/oKtC1sv58ySQ==" sparkle:edSignature="ify59pDIuduaZcLnLvQjGqNQIAqi4dVgeA3L/e7I7xaqn9pVdiVZH7Na3v+Gp4ElAKJfX4Pfq8cgElfXmZc4Cg==" length="1879795" />
```

Once all of your users are running a version of your application with these new updates, you can then ship a new update that drops the DSA public keys and signatures in your appcast. Your new appcast item enclosure may look like:

```xml
<enclosure url="http://sparkle-project.org/myupdate.zip" sparkle:version="2" type="application/octet-stream" sparkle:edSignature="ify59pDIuduaZcLnLvQjGqNQIAqi4dVgeA3L/e7I7xaqn9pVdiVZH7Na3v+Gp4ElAKJfX4Pfq8cgElfXmZc4Cg==" length="1879795" />
```

You may be able to [upgrade using new appcasts](/documentation/publishing#upgrading-to-newer-features) to expedite this process.
