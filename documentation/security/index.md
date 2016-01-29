---
layout: documentation
id: documentation
title: Security
---
### HTTP MITM vulnerability

All Sparkle versions older than 1.13.1 which fetch appcast or release notes over insecure HTTP connection are vulnerable to a man-in-the-middle attack that can lead to disclosure of local files or remote code execution.

Applications using Sparkle with **HTTPS appcast feed URLs and HTTPS release notes links (if any) are safe**.

The vulnerability is [fixed in version 1.31.1](https://github.com/sparkle-project/Sparkle/releases/tag/1.13.1). Patches for older versions are available: [a6e9c](https://github.com/sparkle-project/Sparkle/commit/a6e9c8aff644f0cf5314c9f10e039c34cd350561),
[70f69](https://github.com/sparkle-project/Sparkle/commit/70f6929ac766b404e8e0d28d5cbda7872dc2ee3f).

Thanks to [Radoslaw Karpowicz](https://vulnsec.com) for reporting the vulnerabilty. 


### Bug bounty program

We're going to award a bug bounty for this bug, and we'll be starting a bug bounty program. We'll announce the details soon.

If you've found a vulnerability affecting Sparkle please report it to pornel@pornel.net.
