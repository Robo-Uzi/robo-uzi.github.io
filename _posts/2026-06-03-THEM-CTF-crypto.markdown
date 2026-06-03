---
layout: post
title:  "Crypto Challenges"
date:   2026-06-03 05:46:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THEM-CTF-2026-part2-crypto/
---
* TOC
{:toc}

## D3Spacito

**Category**: Crypto

**Author**: F3L0N

**Description**: lowkey despacito....

I get the challenge files `ques.py` and `output.txt`. Contents of `ques.py`:
```python
from Crypto.Cipher import DES
import base64
from FLAG import flag


def pad(plaintext):
    while len(plaintext) % 8 != 0:
        plaintext += b"*"
    return plaintext

def enc(plaintext, key):
    cipher = DES.new(key, DES.MODE_ECB)
    return base64.b64encode(cipher.encrypt(plaintext))


key = "E1E1E1E1F0F0F0F0".encode().decode("utf-8")
key = bytes.fromhex("E1E1E1E1F0F0F0F0")
plaintext = pad(flag)
print(enc(plaintext, key).decode())
```

`output.txt`:
```shell
cat output.txt  
T/tGpZNyHdhnf1oxwRmMPFcLiH//AfZdTpmYdp8daU0=
```

Solve script:
```shell
python3  
Python 3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> from Crypto.Cipher import DES  
... import base64  
...  
... key = bytes.fromhex("E1E1E1E1F0F0F0F0")  
... ciphertext_b64 = "T/tGpZNyHdhnf1oxwRmMPFcLiH//AfZdTpmYdp8daU0="  
... ciphertext = base64.b64decode(ciphertext_b64)  
... cipher = DES.new(key, DES.MODE_ECB)  
... plain_padded = cipher.decrypt(ciphertext)  
... flag = plain_padded.rstrip(b'*').decode()  
... print(flag)  
...  
THEM?!CTF{D3S_4774K_W3S_AW3S0M3}  
>>> exit
```

The key is `E1E1E1E1F0F0F0F0` (hex, exactly 8 bytes). The ciphertext was given as base64. Decode it to get the raw bytes. They use ECB mode which encrypts each 8 byte block independently. The original `pad()` function added asterisks to make the plaintext length a multiple of 8. After decryption `rstrip(b'*')` removes them. The result is the flag!! 

`THEM?!CTF{D3S_4774K_W3S_AW3S0M3}`

___

## No 7race

**Category**: Crypto

**Author**: F3L0N

**Description**: Can you recover what was lost?

I get the challenge file `challenge.py`:
```python
import sys
sys.set_int_max_str_digits(1000000)

flag = open("flag.txt","rb").read()
if len(flag) > 50:
    exit()

a = int.from_bytes(open("flag.txt","rb").read(), byteorder='big')

b = a << 77777
b = str(b)
if not b.endswith('03081127692533913997381228658418928780421416188103339458770036280397929450297959557812089439331054492922876854076547798835969658432397983993314299716042752'):
    exit()
```

Solve script:
```python
import sys
sys.set_int_max_str_digits(1000000)

suffix_str = (
    "030811276925339139973812286584189287804214161881033394587700362803979"
    "294502979595578120894393310544929228768540765477988359696584323979839"
    "93314299716042752"
)
S = int(suffix_str)

# modulus 5^155
M = 5 ** 155

exponent = 77777 - 155
inv_pow2 = pow(2, -exponent, M)
r = ((S >> 155) * inv_pow2) % M

# known prefix and suffix
prefix = b"THEM?!CTF{"
prefix_int = int.from_bytes(prefix, 'big')
suffix_byte = ord('}')

# try all possible total lengths
found = None
for N in range(len(prefix) + 1, 51):
    L = N - len(prefix) - 1
    if L < 0:
        continue

    term = (prefix_int << (8 * (L + 1))) + suffix_byte

    C = (r - term) % M
    inv256 = pow(256, -1, M)
    X_int = (C * inv256) % M

    if X_int < 256 ** L:
        X_bytes = X_int.to_bytes(L, 'big')
        flag = prefix + X_bytes + b'}'
        a = int.from_bytes(flag, 'big')
        b = str(a << 77777)
        if b.endswith(suffix_str):
            found = flag
            break

if found:
    print(found.decode())
else:
    print("Flag not found")
```

I'm not great at crypto so this is slopped. This is why it works:
```shell
The key insight is that the modulus 5^155 is large enough to make the congruence act as an exact equality for the unknown integer X_int, because the possible range of X_int is much smaller than the modulus. This turns a huge search space into a simple linear equation.
```

`THEM?!CTF{NUMB3R_TH30R3M_1S_FUN}`
