---
layout: post
title:  "Forensics Challenges"
date:   2026-03-30 20:37:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /TexSawCTF-2026-forensics/
---

## A Different Side Channel

**Author**: Written by ag3nt

**Category**: forensics

**Description**: You've captured 500 power traces from a hardware AES encryption device while it processed known plaintexts with an unknown secret key. Flag format: texsaw{flag} ex: texsaw{flag}

I get the challenge files:
```shell
ls  
encrypted_flag.bin  plaintexts.npy  traces.npy

file *  
encrypted_flag.bin: data  
plaintexts.npy:     NumPy data file, version 1.0, description {'descr': '|u1', 'fortran_order': False, 'shape': (500, 16), }
traces.npy:         NumPy data file, version 1.0, description {'descr': '<f8', 'fortran_order': False, 'shape': (500, 100), }
```

To understand the files I process them with numpy:
```python
import numpy as np 
traces = np.load("plaintexts.npy") 
print(np.mean(traces, axis=0))
```

Output:
```shell
[132.298 126.446 120.958 119.1 124.394 125.014 124.942 128.474 128.49 127.76 132.27 130.612 128.526 128.622 125.89 129.514]
```

```python
import numpy as np 
traces = np.load("traces.npy") 
print(np.mean(traces, axis=0))
```

Output:
```shell
[0.48807382 0.51551237 0.50041714 0.49771273 0.50302475 0.50311122 0.4829115 0.50777853 0.49816343 0.49725514 0.49905932 0.49813161 0.49281207 0.4945872 0.51079228 0.51074007 0.50303303 0.49184584 0.50843278 0.97121873 1.9606703 2.55250563 2.56581853 2.55485243 2.56198251 2.56355166 2.60605571 2.59275208 2.5602818 2.561041 2.57452712 2.56541543 2.59392011 2.60723012 2.61421567 2.0963049 1.09663015 0.49721548 0.51252817 0.50142808 0.50934169 0.50867247 0.49717065 0.49709891 0.5066114 0.50078646 0.50553862 0.49093453 0.50190971 0.48678098 0.50587486 0.49510139 0.50307244 0.49126182 0.50825358 0.49951319 0.51218708 0.50513391 0.48489587 0.49501476 0.49674677 0.50087491 0.51249321 0.50451025 0.50064348 0.50089209 0.50212699 0.50697558 0.51195042 0.50874649 0.48968005 0.50434882 0.48799561 0.5120143 0.49658972 0.50467507 0.49536606 0.50509344 0.50070024 0.49794179 0.48046272 0.47709793 0.50381003 0.50636975 0.51941647 0.48551965 0.48835315 0.49958506 0.50354045 0.51612454 0.49624212 0.47636721 0.48881917 0.50075307 0.5016482 0.49700269 0.50745765 0.50130825 0.49894428 0.48972808]
```

Solve script:
```python
import numpy as np
from Crypto.Cipher import AES

plaintexts = np.load("plaintexts.npy")
traces = np.load("traces.npy")
ciphertext = open("encrypted_flag.bin","rb").read()

num_traces, trace_len = traces.shape

# AES SBOX
sbox = [
0x63,0x7c,0x77,0x7b,0xf2,0x6b,0x6f,0xc5,0x30,0x01,0x67,0x2b,0xfe,0xd7,0xab,0x76,
0xca,0x82,0xc9,0x7d,0xfa,0x59,0x47,0xf0,0xad,0xd4,0xa2,0xaf,0x9c,0xa4,0x72,0xc0,
0xb7,0xfd,0x93,0x26,0x36,0x3f,0xf7,0xcc,0x34,0xa5,0xe5,0xf1,0x71,0xd8,0x31,0x15,
0x04,0xc7,0x23,0xc3,0x18,0x96,0x05,0x9a,0x07,0x12,0x80,0xe2,0xeb,0x27,0xb2,0x75,
0x09,0x83,0x2c,0x1a,0x1b,0x6e,0x5a,0xa0,0x52,0x3b,0xd6,0xb3,0x29,0xe3,0x2f,0x84,
0x53,0xd1,0x00,0xed,0x20,0xfc,0xb1,0x5b,0x6a,0xcb,0xbe,0x39,0x4a,0x4c,0x58,0xcf,
0xd0,0xef,0xaa,0xfb,0x43,0x4d,0x33,0x85,0x45,0xf9,0x02,0x7f,0x50,0x3c,0x9f,0xa8,
0x51,0xa3,0x40,0x8f,0x92,0x9d,0x38,0xf5,0xbc,0xb6,0xda,0x21,0x10,0xff,0xf3,0xd2,
0xcd,0x0c,0x13,0xec,0x5f,0x97,0x44,0x17,0xc4,0xa7,0x7e,0x3d,0x64,0x5d,0x19,0x73,
0x60,0x81,0x4f,0xdc,0x22,0x2a,0x90,0x88,0x46,0xee,0xb8,0x14,0xde,0x5e,0x0b,0xdb,
0xe0,0x32,0x3a,0x0a,0x49,0x06,0x24,0x5c,0xc2,0xd3,0xac,0x62,0x91,0x95,0xe4,0x79,
0xe7,0xc8,0x37,0x6d,0x8d,0xd5,0x4e,0xa9,0x6c,0x56,0xf4,0xea,0x65,0x7a,0xae,0x08,
0xba,0x78,0x25,0x2e,0x1c,0xa6,0xb4,0xc6,0xe8,0xdd,0x74,0x1f,0x4b,0xbd,0x8b,0x8a,
0x70,0x3e,0xb5,0x66,0x48,0x03,0xf6,0x0e,0x61,0x35,0x57,0xb9,0x86,0xc1,0x1d,0x9e,
0xe1,0xf8,0x98,0x11,0x69,0xd9,0x8e,0x94,0x9b,0x1e,0x87,0xe9,0xce,0x55,0x28,0xdf,
0x8c,0xa1,0x89,0x0d,0xbf,0xe6,0x42,0x68,0x41,0x99,0x2d,0x0f,0xb0,0x54,0xbb,0x16
]

# Hamming weight lookup
hw = [bin(x).count("1") for x in range(256)]

key = []

for byte_pos in range(16):

    best_corr = 0
    best_guess = 0

    for guess in range(256):

        hypothetical = []

        for t in range(num_traces):
            pt = plaintexts[t][byte_pos]
            val = sbox[pt ^ guess]
            hypothetical.append(hw[val])

        hypothetical = np.array(hypothetical)

        for sample in range(trace_len):

            corr = np.corrcoef(hypothetical, traces[:,sample])[0,1]

            if abs(corr) > best_corr:
                best_corr = abs(corr)
                best_guess = guess

    key.append(best_guess)
    print(f"Byte {byte_pos}: {best_guess:02x}")

key = bytes.fromhex("66dce15fb33deacb5c0362f30e95f52e")

iv = ciphertext[:16]
ct = ciphertext[16:]

cipher = AES.new(key, AES.MODE_CBC, iv)
flag = cipher.decrypt(ct)

print(flag)
```

Output:
```shell
python3 solve1.py  
Byte 0: 66  
Byte 1: dc  
Byte 2: e1  
Byte 3: 5f  
Byte 4: b3  
Byte 5: 3d  
Byte 6: ea  
Byte 7: cb  
Byte 8: 5c  
Byte 9: 03  
Byte 10: 62  
Byte 11: f3  
Byte 12: 0e  
Byte 13: 95  
Byte 14: f5  
Byte 15: 2e  
b'texsaw{d1ffer3nti&!_p0w3r_@n4!y51s}\r\r\r\r\r\r\r\r\r\r\r\r\r'
```

`texsaw{d1ffer3nti&!_p0w3r_@n4!y51s}`
