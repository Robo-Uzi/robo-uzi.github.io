---
layout: post
title:  "Dark Tracers"
date:   2026-01-12 01:23:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /scarletCTF-dark-tracers/
---

**Title**: Dark Tracers

**Category**: Forensics

**Author**: glqce

**Description**: Would you look at that!? We have a fake murder-for-hire case on our desk. Lovely. Anyway, ASAC Bobr wants you to trace through the transaction to identify the most likely hash that indicates the payment made from the perp to the scammer for the agreed upon amount.

Here is what we believe is the transaction made from a Bitcoin ATM to one of the addresses associated with the perpetrator's wallet: `427e04420fffc36e7548774d1220dad1d20c1c78dd71ad2e1e9fd1751917a035`

Find the transaction made from the victim to the scammer and upload the transaction hash as the flag!

Here is the press release for the case, a young agent like you always needs some good old [context](https://www.justice.gov/usao-ndtx/pr/woman-sentenced-9-years-dark-web-murder-hire-plot)
FLAG FORMAT: `RUSEC{hash}`

For example, if you think the transaction hash is `49100341fe8d99bd1ed7cec22edf0ec63d6920a01627e34249b2dfb3c464bca0`, upload it in the format: `RUSEC{49100341fe8d99bd1ed7cec22edf0ec63d6920a01627e34249b2dfb3c464bca0}`

From [https://www.justice.gov/usao-ndtx/pr/woman-sentenced-9-years-dark-web-murder-hire-plot](https://www.justice.gov/usao-ndtx/pr/woman-sentenced-9-years-dark-web-murder-hire-plot) I know Im looking for a transaction from July 27th, 2023 for `0.358` bitcoin. 

After clicking around on [https://blockstream.info/](https://blockstream.info/) I found the transaction at [https://blockstream.info/address/bc1q44mw0cffurnex8jxqvtvap3fwv3et0v9lxdc3t](https://blockstream.info/address/bc1q44mw0cffurnex8jxqvtvap3fwv3et0v9lxdc3t)! 

`RUSEC{57ce32d129f4824aa8c7e71e56cf4908dcc32103f5fff3c3d6a08bd7bae78c48}`