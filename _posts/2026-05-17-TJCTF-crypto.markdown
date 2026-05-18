---
layout: post
title:  "Crypto Challenges"
date:   2026-05-17 20:54:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /TJCTF-2026-crypto/
---
* TOC
{:toc}

## wavy

**Author**: ashwathg

**Category**: crypto

**Description**: why does my decode script not work :(

I get the challenge files. `decode.py`:
```python
from pathlib import Path

from Crypto.Util.number import long_to_bytes

M = 0xfffffffffffffffffffffffffffffffffffffffffffffffffffffffefffffc2f


def encrypt(val, key):
    amp = val
    p = M

    prev = 1
    current = amp
    for _ in range(key - 1):
        new_val = (2 * amp * current - prev) % p
        prev = current
        current = new_val
    return current

flag_bytes = ("flag.enc").read_bytes()

base_val = 0x1337C0DE
frequency_key = 10**25

secret = encrypt(base_val, frequency_key)

encrypted = bytes([a ^ b for a, b in zip(flag_bytes, long_to_bytes(secret))])

print(encrypted)
```

`flag.enc`:
```shell
xxd flag.enc  
00000000: 7f9f ef05 31c7 66ba 9748 c07c 5b92 25a6  ....1.f..H.|[.%.  
00000010: 514e b357 6e7d 8339 4451 5289 bfbb d299  QN.Wn}.9DQR.....  
00000020: 3c88 86                                  <..
```

The encryption uses a Chebyshev polynomial `T_n(x)` modulo the prime `p=2^256-2^32-977`. The secret is `T_10^25​(0x1337C0DE)mod p`. The ciphertext is the XOR of the flag bytes with the byte representation of this secret. Solve script:
```python
from pathlib import Path
from Crypto.Util.number import long_to_bytes

M = 0xfffffffffffffffffffffffffffffffffffffffffffffffffffffffefffffc2f

def chebyshev(x, n, mod):
    # return T_n(x) mod mod using matrix exponentiation
    if n == 0:
        return 1
    if n == 1:
        return x % mod
    # matrix [[2x, -1], [1, 0]]
    def mat_mul(A, B):
        return [
            [(A[0][0]*B[0][0] + A[0][1]*B[1][0]) % mod,
             (A[0][0]*B[0][1] + A[0][1]*B[1][1]) % mod],
            [(A[1][0]*B[0][0] + A[1][1]*B[1][0]) % mod,
             (A[1][0]*B[0][1] + A[1][1]*B[1][1]) % mod]
        ]
    def mat_pow(mat, exp):
        res = [[1,0],[0,1]]
        while exp:
            if exp & 1:
                res = mat_mul(res, mat)
            mat = mat_mul(mat, mat)
            exp >>= 1
        return res
    mat = [[(2*x) % mod, (mod-1) % mod], [1, 0]]
    mat_exp = mat_pow(mat, n-1)
    return (mat_exp[0][0] * x + mat_exp[0][1]) % mod

base_val = 0x1337C0DE
frequency_key = 10**25

secret = chebyshev(base_val, frequency_key, M)
key_bytes = long_to_bytes(secret)

ciphertext = Path("flag.enc").read_bytes()
key_stream = (key_bytes * (len(ciphertext) // len(key_bytes) + 1))[:len(ciphertext)]
plaintext = bytes(a ^ b for a, b in zip(ciphertext, key_stream))
print(plaintext.decode())
```

Output:
```shell
python3 solve.py  
tjctf{ch3bysh3v_p0lyn0m1!l_676767}
```

`tjctf{ch3bysh3v_p0lyn0m1!l_676767}`
