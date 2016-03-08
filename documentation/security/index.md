---
layout: documentation
id: documentation
title: Security
---
### SparkleUpdater before 1.13.1 has serious vulnerabilities

TL;DR: use HTTPS for your Appcast Feed to limit the possibility of someone exploiting this vulnerability and upgrade to atleast 1.13.1 to make the vulnerability actually go away. 

Machines running applications built with SparkleUpdater framework older than 1.13.1 are vulnerable to remote code execution and the disclosure of local files. SparkleUpdater allows you to add HTML and JavaScript code to your update description. An attacker could ultimately use this to make SparkleUpdater load and execute commands from a remote server. As SparkleUpdater before 1.13.1 also processed external entities in the Appcast XML Feed this also could be abused to read and exfiltrate local files without executing code.

The full attack is described in detail in the report written by Radoslaw Karpowicz: "[There's a lot of vulnerable OS X applications out there.](https://vulnsec.com/2016/osx-apps-vulnerabilities/)". Extra details about the impact of this issue can be found in our [Github issue that tracks this vulnerability](https://github.com/sparkle-project/Sparkle/issues/722). 

This vulnerability can be exploited by performing a Man-in-the-Middle attack when you use HTTP as your Appcast Feed URL but also by someone gaining write access to the Appcast Feed on your webserver (e.g. when your webserver is hacked). A Proof-of-Concept attack that can [automatically attack vulnerable applications on a shared network](https://www.evilsocket.net/2016/01/30/osx-mass-pwning-using-bettercap-and-the-sparkle-updater-vulnerability/) was made available by Simone Margaritelli.  

Thanks to [Radoslaw Karpowicz](//vulnsec.com) for reporting the vulnerabilty.

## Mitigations

* Upgrade to atleast [SparkleUpdater framework 1.13.1]((//github.com/sparkle-project/Sparkle/releases/tag/1.13.1). This version enhances security by further restricting permissions for the webview used in the update dialog and disables external entity parsing all together. These changes protect your users from both the RCE and the info-disclosure vulnerabilities. 
* If you are unable to update to 1.13.1 for some reason. Patches for older versions of Sparkle are available: [a6e9c](//github.com/sparkle-project/Sparkle/commit/a6e9c8aff644f0cf5314c9f10e039c34cd350561), [70f69](//github.com/sparkle-project/Sparkle/commit/70f6929ac766b404e8e0d28d5cbda7872dc2ee3f). If you use any older versions of Sparkle, or forks of the official version, please verify that they have these patches applied.
* Make sure your application loads the Appcast Feed over HTTPS only. You can use letsencrypt to [get a free X509 certificate](https://letsencrypt.org/). This makes it harder to exploit applications running older versions of Sparkle but doesn't fully protect your users. 

### Bug bounty program

We've awarded a $250 bounty for Radoslaw's report. We're starting an official bug bounty program. We'll announce the details soon.

If you've found a vulnerability affecting Sparkle please report it to pornel@pornel.net.
