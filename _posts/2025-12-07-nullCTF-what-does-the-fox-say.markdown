---
layout: post
title:  "What does the fox say?"
date:   2025-12-07 16:17:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /nullCTF-what-does-the-fox-say/
---
* TOC
{:toc}

**Title**: What does the fox say?

**Category**: misc

**Author**: cshark3008

**Description**: The fox left behind a strange melody... Hidden in the noise lies the truth. Can you figure out what the fox says? [🦊](https://www.youtube.com/watch?v=jofNR_WkoCE) [http://public.ctf.r0devnull.team:3012](http://public.ctf.r0devnull.team:3012)

`http://public.ctf.r0devnull.team:3012/robots.txt`:
```shell
User-agent: *
Disallow: /secret.txt
```

`http://public.ctf.r0devnull.team:3012/secret.txt`:
```shell
The fox whispers a secret hidden in the noise... Can you hear it?
G44TUJJQIZLEU5DKHZTSONDNKRMF2YATGJ4D2VLPKUBDO5LFHNRFYVYCK5UCEMJU
```

base32 decode then XOR with repeating key `YLVIS2013`:
```python
import base64
s = "G44TUJJQIZLEU5DKHZTSONDNKRMF2YATGJ4D2VLPKUBDO5LFHNRFYVYCK5UCEMJU"
data = base64.b32decode(s)
key = b"YLVIS2013"
out = bytes([data[i] ^ key[i % len(key)] for i in range(len(data))])
print(out.decode())
```

`nullctf{G3r1ng_din9_d1ng_d1n93r1ng3d1ng}`