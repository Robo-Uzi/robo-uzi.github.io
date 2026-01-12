---
layout: post
title:  ":("
date:   2026-01-12 01:23:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /scarletCTF-frowny-face/
---

**Title**: :(

**Category**: Forensics

**Author**: ilyree

**Description**: As I look back at my RUSEC memories, I remembered the time that I met my mentor! Seems like he accidently kept sending my machine a payload that made my screen go blue...

I downloaded the challenge file (a windows event log):
```shell
file sad_face.zip  
sad_face.zip: Zip archive data, at least v2.0 to extract, compression method=deflate

unzip sad_face.zip  
Archive:  sad_face.zip  
 inflating: Challenge.evtx
 
file Challenge.evtx  
Challenge.evtx: MS Windows 10-11 Event Log, version  3.2, 3 chunks (no. 2 in use), next record no. 431
```

While trying some basic things I tried grepping for base64:
```shell
strings Challenge.evtx | grep "=="  
BppCTUE1i3825UwqP9UDaj71NdSXowxSPcI/XA==  
TED5ocxPeiQziOrTQsJhAna42gc/iUkaHVEaRg==  
BHX01ANtcdylDobfffrdzM3Cm50BrcxPTqso4A==  
vKnvBzqLaJQXlCLsuDNYliTMWzPD0U6FgAY1eg==  
SRurKI6HYICisSSHf+g7YwWFP9CpkgkXLjoCyg==  
Ak+mW8WlVzgUN7+UuiC2LVyPAGZstoxNYJUQZA==  
jsFhexmhTyWeVcEwgdWa+j1I5ANSeEbfDsndtA==  
R/Zdr1C/Rt0Q2108vA0VxJRSpZkUnMkVQCTrTg==  
UlVTRUN7M3Rlcm5hbF9ibHUzXw==  
jJwTAtvZNYENfvvkv/p0XMwVSsy8ggbdILPFOA==  
c0BkX2ZhYzNfc21idg==  
MV8zODkwY24yazI5fQ==  
BppCTUE1i3825UwqP9UDaj71NdSXowxSPcI/XA==  
TED5ocxPeiQziOrTQsJhAna42gc/iUkaHVEaRg==  
BHX01ANtcdylDobfffrdzM3Cm50BrcxPTqso4A==  
vKnvBzqLaJQXlCLsuDNYliTMWzPD0U6FgAY1eg==  
SRurKI6HYICisSSHf+g7YwWFP9CpkgkXLjoCyg==  
Ak+mW8WlVzgUN7+UuiC2LVyPAGZstoxNYJUQZA==  
jsFhexmhTyWeVcEwgdWa+j1I5ANSeEbfDsndtA==  
R/Zdr1C/Rt0Q2108vA0VxJRSpZkUnMkVQCTrTg==  
UlVTRUN7M3Rlcm5hbF9ibHUzXw==  
jJwTAtvZNYENfvvkv/p0XMwVSsy8ggbdILPFOA==  
c0BkX2ZhYzNfc21idg==  
MV8zODkwY24yazI5fQ==
```

After putting all these strings into cyberchef and base64 decoding I found the flag in two parts. 

I did download `evtx_dump` to see if I could find a diferent way to get the flag. I couldnt. I assume eternal blue was causing the BSOD.
```shell
./evtx_dump Challenge.evtx | wc -l  
12900
```

`RUSEC{3ternal_blu3_s@d_fac3_smbv1_3890cn2k29}`