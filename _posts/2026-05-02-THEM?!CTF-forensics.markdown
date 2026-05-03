---
layout: post
title:  "Forensics Challenge"
date:   2026-05-02 18:59:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THEMCTF-2026-forensics/
---

CTF got hacked :( I dont have the title or author of this challenge.

```shell
unzip phantom.zip  
Archive:  phantom.zip  
   creating: logs/  
   creating: media/  
   creating: signals/  
  inflating: net.pcap  
  inflating: ram.raw  
  inflating: disk.img  
  inflating: logs/timeline_A.log  
  inflating: logs/timeline_B.log  
  inflating: logs/observer.log  
  inflating: media/frame_001.png  
  inflating: media/frame_002.png  
  inflating: media/transmission.wav
```

There are many fake flags.

The logs:
```shell
ls  
observer.log  timeline_A.log  timeline_B.log  

cat observer.log  
[observer] Observation collapses the signal.  
[observer] If two sources disagree, trust neither.  
[observer] Truth is what survives comparison.  
[observer] Network: compare packet lanes by sequence id.  
[observer] Memory: mirrored regions begin after REALITY_A_BEGIN / REALITY_B_BEGIN.  
[observer] Disk: the deleted shards disagree by design.  
[observer] Signal: one payload is not on disk at all.  

cat timeline_A.log  
[2026-01-11 21:44:02] recovery pipeline booted  
[2026-01-11 21:44:03] observer sync: disabled  
[2026-01-11 21:44:05] recovered flag candidate => THEM?!CTF{reality_is_A}  
[2026-01-11 21:44:08] tx packet lane A exported  
[2026-01-11 21:44:13] ram mirror A sealed  
[2026-01-11 21:44:21] disk shard A archived  
[2026-01-11 21:44:34] note: forward signal looked clean  

cat timeline_B.log  
[2026-01-11 21:44:02] recovery pipeline booted  
[2026-01-11 21:44:03] observer sync: enabled  
[2026-01-11 21:44:05] recovered flag candidate => THEM?!CTF{reality_is_B}  
[2026-01-11 21:44:08] tx packet lane B exported  
[2026-01-11 21:44:13] ram mirror B sealed  
[2026-01-11 21:44:21] disk shard B archived  
[2026-01-11 21:44:34] note: reverse signal looked clean
```

Looks like there is 4 flag parts. One in `net.pcap`, one in `ram.raw`, one in `disk.img`, and one in `transmission.wav` or one of the images.

In the pcap:
```shell
tshark -r net.pcap -T fields -e tcp.seq -e data | while read seq data; do  
echo "$seq $(echo $data | xxd -r -p)"  
done  
1  
1 01|mk3mn0  
1 01|mw3my0  
101 02|zry_w1  
101 02|ery_h1  
201 03|s3_  
201 03|sq_  
7374617465  
6167726565  
3d3d  
7472757468

mk3mn0zry_w1s3_

mw3my0ery_h1sq_

# diff
m3m0ry_1s_
```

In `ram.raw`:
```shell
strings ram.raw | grep '_'  
_dC`|  
PROC:ai_core.bin  
THEM?!CTF{memory_strings_are_lying}  
REALITY_A_BEGIN  
n0td_suwh4t_l1dt_  
REALITY_A_END  
"_q=/  
REALITY_B_BEGIN  
n0tb_mawh4t_816t_  
REALITY_B_END  
_(w9  
.}_u

n0td_suwh4t_l1dt_

n0tb_mawh4t_816t_

# diff
n0t_wh4t_1t_
```

In `disk.img`:
```shell
strings * | grep "_"
... etc
_t<F  
P-N_{k  
Xj_!  
FRAGMENT_A=7s33ms_2y0cu_3  
FRAGMENT_B=5s33ms_9y02u_w  
THEM?!CTF{easy_disk_flag}  
.'_F  
a_o''z  
_<1~d0
... etc

7s33ms_2y0cu_3

5s33ms_9y02u_w

# diff
s33ms_y0u_
```

The audio file has morse code and a hint in lsb. The morse code:
```shell
.- .-. . / -.-- --- ..- / --- -.-

# decode
ARE YOU OK
```

lsb of `transmission.wav`:
```shell
python3  
Python 3.13.5 (main, Jun 25 2025, 18:55:22) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> from scipy.io import wavfile  
... import numpy as np  
...  
... rate, data = wavfile.read("transmission.wav")  
...  
... if len(data.shape) > 1:  
...     data = data[:, 0]  
...  
... bits = data & 1  
...  
... out = []  
... for i in range(0, len(bits), 8):  
...     byte = bits[i:i+8]  
...     if len(byte) < 8:  
...         break  
...     value = int("".join(map(str, byte)), 2)  
...     out.append(value)  
...  
... result = bytes(out)  
...  
... print(result[:20])  
...  
b'\x00\x12lsb->frame_002.png'
```

Out of `frame_001.png` and `frame_002.png` I will focus on 2.

Extract zip file out of image:
```shell
python3  
Python 3.13.5 (main, Jun 25 2025, 18:55:22) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> from PIL import Image  
... import numpy as np  
...  
... img2 = np.array(Image.open('frame_002.png'))  
...  
... # extract all lsbs from all channels  
... all_lsbs = (img2 & 1).flatten()  
... bits = ''.join(map(str, all_lsbs))  
...  
... # convert bits to bytes  
... n_bytes = len(bits) // 8  
... raw = bytes([int(bits[i*8:(i+1)*8], 2) for i in range(n_bytes)])  
...  
... # find the PK (zip) header  
... pk_pos = raw.find(b'PK')  
... print(f"PK found at byte: {pk_pos}")  
... print(f"First bytes: {raw[:4].hex()}")  
...  
... # save from PK on  
... zip_data = raw[pk_pos:]  
... with open('hidden.zip', 'wb') as f:  
...     f.write(zip_data)  
... print(f"Saved {len(zip_data)} bytes to hidden.zip")  
...  
PK found at byte: 4  
First bytes: 0000011b  
Saved 98300 bytes to hidden.zip

file hidden.zip  
hidden.zip: Zip archive data, at least v2.0 to extract, compression method=deflate

# zip incomplete (i think it ends wrong) but I can unzip in the file manager. i get these files:
ls  
fragment_4.txt  readme.txt  

cat readme.txt  
If you found me, compare realities everywhere.  

cat fragment_4.txt  
4r3_cl0s3
```

CTF is hacked so I cant check the flag. Is it:

`THEM?!CTF{s33ms_y0u_m3m0ry_1s_n0t_wh4t_1t_4r3_cl0s3?}` ??? 

I think. 

