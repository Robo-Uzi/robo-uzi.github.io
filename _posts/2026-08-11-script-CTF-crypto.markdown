---
layout: post
title:  "Crypto Challenges"
date:   2026-08-11 20:57:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /script-CTF-2026-crypto/
---
* TOC
{:toc}

## Misdirection

**Category**: crypto

**Author**: NoobMaster

**Description**: It is not what it is.

I get the challenge file:
```shell
cat enc.txt  
1000100010100000100001110100100001010010001010110001101100101010000111000001001001000100101000100100001000101110001
```

I decoded from `Bacon Cipher` on cyberchef: [cyberchef link](https://gchq.github.io/CyberChef/#recipe=Bacon_Cipher_Decode('Standard%20(I%3DJ%20and%20U%3DV)','0/1',false\)&input=MTAwMDEwMDAxMDEwMDAwMDEwMDAwMTExMDEwMDEwMDAwMTAxMDAxMDAwMTAxMDExMDAwMTEwMTEwMDEwMTAxMDAwMDExMTAwMDAwMTAwMTAwMTAwMDEwMDEwMTAwMDEwMDEwMDAwMTAwMDEwMTExMDAwMQo)

`SCRIPTCTF{NOTWHATITSEEMS}`

___

## Oops

**Category**: crypto

**Author**: NoobMaster

**Description**: I am from the future! I accidentally forgot to link `chall.zip`! Surely you can find it and solve it right?

They didnt link the download for the challenge file so I had to get the link for myself. I looked at the format of the other download links and adjusted it:
```shell
https://scriptctf-2026-wave1-randomchars-<redacted-tbh>.s3.us-east-1.amazonaws.com/Crypto/Oops/chall.zip
```

Now I have the challenge files:
```shell
unzip chall.zip  
Archive:  chall.zip  
  inflating: chall.py  
replace enc.txt? [y]es, [n]o, [A]ll, [N]one, [r]ename: r  
new name: enc2.txt  
  inflating: enc2.txt
```

Looking at the challege files:
```shell
cat enc2.txt  
d37cbce47f0c71a75d644badb77039e48ab1645f60ddebe928c0a3c417561345b4852636ecb388ec79417357100da120
```

`chall.py`:
```python
import random
import time
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
from hashlib import sha256

flag = open('flag.txt','rb').read()

random.seed(int(time.time())) # Preserves upto the MINUTE, not seconds ;)
key = random.randbytes(32)

cipher = AES.new(key, AES.MODE_ECB)

enc = cipher.encrypt(pad(flag,16)).hex()

open('enc.txt', 'w').write(enc)
```

Pythons `random` module is a PRNG. Calling `random.seed(seed)` always produces the same sequence of bytes. The seed used for encryption is the current Unix timestamp so it is pretty easy to brute force. 

Solve script:
```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import random
import os
import time

ct = bytes.fromhex("d37cbce47f0c71a75d644badb77039e48ab1645f60ddebe928c0a3c417561345b4852636ecb388ec79417357100da120")
mtime = int(os.path.getmtime('enc2.txt'))
for delta in range(-3600, 3601):
    seed = mtime + delta
    random.seed(seed)
    key = random.randbytes(32)
    cipher = AES.new(key, AES.MODE_ECB)
    pt = cipher.decrypt(ct)
    try:
        pt = unpad(pt, 16)
        if pt.startswith(b'scriptCTF{'):
            print(seed, pt)
            break
    except ValueError:
        pass
```

Output:
```shell
python3 solve.py  
3153037173 b'scriptCTF{mY_buck37_1s_l34k1ng!}\n'
```

`scriptCTF{mY_buck37_1s_l34k1ng!}`
