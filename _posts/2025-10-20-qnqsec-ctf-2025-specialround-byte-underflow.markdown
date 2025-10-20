---
layout: post
title:  "Byte-Underflow"
date:   2025-10-20 03:07:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-special-round-byte-underflow/
---

**Title:** Byte-Underflow

**Category:** stego

**Description:** Something tells us this picture used to hold a secret — but someone decided to “shift the brightness down” for safekeeping. You are given a single file: flag.jpg. Your task: recover the original image and extract the flag hidden in it. Flag format: byteshift{...}

**Author:** Winterflake

I get the file `flag.jpg`:
```shell
file flag.jpg  
flag.jpg: data
```
It doesnt have a recognized file format just data.

When I run strings I see a flag but it is a decoy:
```shell
strings flag.jpg | tail  
bnbc  
zFl]  
#sO*  
W-"X    
VJD5W  
8S;m  
<yU^  
p)xn$  
lO60  
'byteshift{wow_this_byteshift_is_not_secure_at_all}'
```

I run this script to recover the flag:
```python
with open('flag.jpg', 'rb') as f:
    data = f.read()
fixed = bytes((b + 1) % 256 for b in data)
with open('original.jpg', 'wb') as f:
    f.write(fixed)
print("Image saved as original.jpg.")
```
As it is mentioned, the bytes in the photo were shifted. The script applies the inverse operation by incrementing each byte by 1 modulo 256 (to handle the wrap around for `0xFF` becoming `0x00`). 

Final image:

![Alt text](/images/original.jpg)

`byteshift{wow_can_you_bel1eve_that_1_byt3_w4s_shifted_2_times}`