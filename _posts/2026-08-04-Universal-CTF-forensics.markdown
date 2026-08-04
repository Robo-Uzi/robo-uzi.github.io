---
layout: post
title:  "Forensics Challenges"
date:   2026-08-04 18:10:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Universal-CTF-2026-forensics/
---
* TOC
{:toc}
## Paper Jam

**Author**: Yewolf

**Category**: forensics

**Description**: A berth office multifunction printer tried to export a scanned manifest during a power fault. The resulting PDF is damaged. The office insists the scan completed, but the document won't open in standard viewers. Recover the document and extract the authorization token.

I get the challenge file:
```shell
file shipping_notice.pdf  
shipping_notice.pdf: data  

xxd shipping_notice.pdf | head  
00000000: 2550 3046 2d31 2e34 0a25 e2e3 cfd3 0a31  %P0F-1.4.%.....1  
00000010: 2030 206f 626a 0a3c 3c20 2f54 7970 6520   0 obj.<< /Type  
00000020: 2f43 6174 616c 6f67 202f 5061 6765 7320  /Catalog /Pages  
00000030: 3220 3020 5220 3e3e 0a65 6e64 6f62 6a0a  2 0 R >>.endobj.  
00000040: 3220 3020 6f62 6a0a 3c3c 202f 5479 7065  2 0 obj.<< /Type  
00000050: 202f 5061 6765 7320 2f4b 6964 7320 5b33   /Pages /Kids [3  
00000060: 2030 2052 2036 2030 2052 5d20 2f43 6f75   0 R 6 0 R] /Cou  
00000070: 6e74 2032 203e 3e0a 656e 646f 626a 0a33  nt 2 >>.endobj.3  
00000080: 2030 206f 626a 0a3c 3c20 2f54 7970 6520   0 obj.<< /Type  
00000090: 2f50 6167 6520 2f50 6172 656e 7420 3220  /Page /Parent 2
```

I noticed `P0F` and changed it to `PDF`. This caused the file type to be recognized but did not render any content in the pdf. I put the corrupted pdf into [https://www.ilovepdf.com/repair-pdf](https://www.ilovepdf.com/repair-pdf) and it repaired the file!

The repaired pdf contained an image with the flag:

![Alt text](/images/uniforensics1.png)

`uctf{9f2d7b4c6a81e305}`
