---
layout: post
title:  "Exponope"
date:   2026-05-12 19:34:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THCON-CTF-2026-exponope/
---

**Category**: Crypto

**Author**: Aurélien Pouilles ( credit goes to Nathan Maillet for the original challenge). This challenge was created by a TLS-SEC student

**Description**: Our cryptography expert, Axel Vaughn just announced he's implemented a Secure way of encryption that is very efficient in order to lower the latency of our troops' telecommunications.

To do so he lowered the exponent used to a tiny value and claims the security impact is negligible. He sent us the following file for proof. Can you try to crack it ?

I get the challenge file:
```shell
cat vewy-much-mysterious-file-such-encryptationnly-encrypted.crypt  
N = 0x6361956d13a35165056e1d17a7823cff3579b81964bef82da624425f15722a8dbd15ecbf0fbcaa3a386eb42af26994377d8c77d5fc80fc351186441429ea563a365304d6832a3ac2c5ff911684abe3f6a531da92260a6ed856e543ed723c20124d368699c7b95b3c46927d3ead9123ea6f6950f19090318d98e0a499535b9f2f20fe08a9998c65346e44b3c2b86adf952282816d7daeb5823e0c3ac7a732d7a28deaf934bdc8de1f6a598eaf7d1a6942590da278c7f72f2183593b6ca48aae09efd4acf9b2e4d768ea37388356e913f0142392dbd5c4ccd484ee09fbf67e9f279601fe18eba58fdc7e2b9a8328dcb50fca38a7e8da894b65521b07fb89d3d25d  
cyphertext = 0xfd7d893b965ca58d3e19e07e85f95b440b3e66245a14ad601c8aeae7b139b3d044ec05e9bd4e1ba9a9f2603d4d12bbd343068e55d454417a9ef0dd4d8c8deb717de0c538ca56524ce3e0adee6542d5b710ca6359510d05b9e04fd49adf8f2cf63a7eda6d69b6a59982311801e48c6a5a0154c5bc206dcf7315441d838859871a444cd81c4836b660f26a5e69a7702d8d
```

This is RSA with a small exponent:
```python
from sympy import integer_nthroot

N = int("6361956d13a35165056e1d17a7823cff3579b81964bef82da624425f15722a8dbd15ecbf0fbcaa3a386eb42af26994377d8c77d5fc80fc351186441429ea563a365304d6832a3ac2c5ff911684abe3f6a531da92260a6ed856e543ed723c20124d368699c7b95b3c46927d3ead9123ea6f6950f19090318d98e0a499535b9f2f20fe08a9998c65346e44b3c2b86adf952282816d7daeb5823e0c3ac7a732d7a28deaf934bdc8de1f6a598eaf7d1a6942590da278c7f72f2183593b6ca48aae09efd4acf9b2e4d768ea37388356e913f0142392dbd5c4ccd484ee09fbf67e9f279601fe18eba58fdc7e2b9a8328dcb50fca38a7e8da894b65521b07fb89d3d25d", 16)

c = int("fd7d893b965ca58d3e19e07e85f95b440b3e66245a14ad601c8aeae7b139b3d044ec05e9bd4e1ba9a9f2603d4d12bbd343068e55d454417a9ef0dd4d8c8deb717de0c538ca56524ce3e0adee6542d5b710ca6359510d05b9e04fd49adf8f2cf63a7eda6d69b6a59982311801e48c6a5a0154c5bc206dcf7315441d838859871a444cd81c4836b660f26a5e69a7702d8d", 16)

# try small exponents
for e in range(3, 11):
    m, exact = integer_nthroot(c, e)
    if exact:
        print(f"Found e = {e}")
        print(bytes.fromhex(hex(m)[2:]).decode(errors="ignore"))
        break
else:
    print("No exact root found ;(")
```
You can recover `m` by taking an integer `e-th` root of the ciphertext.

Output:
```shell
python3 solve.py  
Found e = 5  
THC{u_n3eD_@_bett3r_eXp0neNT}
```

`THC{u_n3eD_@_bett3r_eXp0neNT}`