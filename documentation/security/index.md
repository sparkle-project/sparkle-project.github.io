---
layout: documentation
id: documentation
title: Security
---
### Vulnerabilities in Sparkle before 1.13.1

All Sparkle versions older than 1.13.1 (including 1.5b6) which fetch appcast or release notes over insecure HTTP connection are vulnerable to [a man-in-the-middle attack that can lead to disclosure of local files or remote code execution](https://vulnsec.com/2016/osx-apps-vulnerabilities/).

The vulnerability is fixed in version [1.13.1](https://github.com/sparkle-project/Sparkle/releases/tag/1.13.1). **[We recommend all developers to update ASAP](https://github.com/sparkle-project/Sparkle/releases/latest)**. For most applications it's as easy as replacing `Sparkle.framework` with the new version ([upgrading notes](/documentation/upgrading/)).

#### Mitigations for developers

* [Update Sparkle](https://github.com/sparkle-project/Sparkle/releases/latest) to at least v1.13.1.
* If you are unable to update for some reason, patches for older versions of Sparkle are available: [a6e9c](https://github.com/sparkle-project/Sparkle/commit/a6e9c8aff644f0cf5314c9f10e039c34cd350561), [70f69](https://github.com/sparkle-project/Sparkle/commit/70f6929ac766b404e8e0d28d5cbda7872dc2ee3f). If you use any older versions of Sparkle, or forks of the official version, please verify that they have these patches applied.
* Make sure your application loads the appcast feed and release notes (if any) over **HTTPS only** ([HTTPS certificates are free from Let's Encrypt](https://letsencrypt.org/)). HTTPS makes the vulnerability much harder to exploit, but we recommend upgrading Sparkle as well as defence-in-depth (e.g. against compromised servers).


Thanks to [Radoslaw Karpowicz](https://vulnsec.com) for reporting the vulnerabilty.

### Bug bounty program

We've awarded a $250 bounty for Radoslaw's report. We're starting an official bug bounty program. We'll announce the details soon.

If you've found a vulnerability affecting Sparkle please report it to pornel@pornel.net.
