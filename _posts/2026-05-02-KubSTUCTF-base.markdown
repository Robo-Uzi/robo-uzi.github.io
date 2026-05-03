---
layout: post
title:  "Base"
date:   2026-05-02 19:17:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /KubSTU-2026-base/
---

**Author**: [@ST47IC4](https://t.me/ST47IC4)

**Category**: Crypto

**Description**: We intercepted a strange message. It seems to be encoded using a popular method. Help us figure out what it says. Flag format: `KubSTU(...)`

I get the challege file:
```shell
cat Base.txt  
S3ViU1RVKGI0czNfNjRfMXNfdGhlX2JhNWk1KQ==
```

Simply `base64` decode:
```shell
echo "S3ViU1RVKGI0czNfNjRfMXNfdGhlX2JhNWk1KQ==" | base64 -d  
KubSTU(b4s3_64_1s_the_ba5i5)
```

`KubSTU(b4s3_64_1s_the_ba5i5)`