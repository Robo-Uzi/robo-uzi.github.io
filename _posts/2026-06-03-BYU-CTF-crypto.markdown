---
layout: post
title:  "Crypto Challenges"
date:   2026-06-03 05:46:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /BYU-CTF-2026-crypto/
---
* TOC
{:toc}

**Title**: RSA Dreams

**Category**: Crypto

**Author**: wywy213

**Description**: Did I give one too many hints????

I get the challenge files:
```shell
unzip rsa_dreams_dist.zip  
Archive:  rsa_dreams_dist.zip  
  inflating: output.txt  
  inflating: encrypt.py
```

`output.txt`:
```shell
cat output.txt  
c = 5074616349947930347771128443869249667723941019037011379843932659330729580197593522845748676276068149443504037403162962894625104017380398816152166168830833  
n = 6452268004013779272669102227661703532150635430524568657091997086066784917218113937677647594597481724133073200003104968955173212323278046973034541033497147  
e = 65537  
hint = 161697499284577475400347684012866511237569864647822807778480533925514941939388
```

`encrypt.py`:
```python
from Crypto.Util.number import getPrime
from Crypto.Util.number import bytes_to_long

flag = b"REDACTED"

p = getPrime(256)
q = getPrime(256)
n = p*q
hint = p + q
e = 0x10001

c = pow(bytes_to_long(flag), e, n)

print(f"c = {c}")
print(f"n = {n}")
print(f"e = {e}")
print(f"hint = {hint}")
```

Solve script:
```python
from Crypto.Util.number import long_to_bytes
from math import isqrt

c = 5074616349947930347771128443869249667723941019037011379843932659330729580197593522845748676276068149443504037403162962894625104017380398816152166168830833
n = 6452268004013779272669102227661703532150635430524568657091997086066784917218113937677647594597481724133073200003104968955173212323278046973034541033497147
e = 65537
s = 161697499284577475400347684012866511237569864647822807778480533925514941939388

disc = s * s - 4 * n
r = isqrt(disc)

p = (s + r) // 2
q = (s - r) // 2

phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)

m = pow(c, d, n)
print(long_to_bytes(m).decode())
```

Normally RSA is secure because factoring n into p and q is hard. But knowing both the sum and product of two numbers lets you recover them directly. Once you have p and q RSA is broken. 

`byuctf{great_job_recovering_the_flag}`
