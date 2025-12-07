---
layout: post
title:  "Classically"
date:   2025-12-07 16:37:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /nullCTF-classically/
---

**Title**: Classically

**Category**: crypto

**Author**: infernosalex

**Description**: "Do you think you can solve this classically?" Flag format: ctf{}

I get the challenge files `M.py` and `main.py`:
```shell
cat M.py  
"""Statically defined 64x64 matrix for the linear system."""  
  
M = [  
   [57121, 19227, 32536, 277, 59722, 59191, 63981, 65014, 21161, 58022, 20206, 57613, 62723, 57302, 43570, 52022, 39155, 8180, 53332, 26348, 10715, 35123, 51712, 31978, 42383, 1938, 5076, 57300, 60128, 8229, 39370, 48263, 15527, 46890, 21181, 37520, 12962, 43620, 58636, 61383, 9476, 15805, 9435, 12791, 65295, 39577, 49230, 60291, 32630, 35471, 51322, 53976, 30361, 14514, 14231, 27362, 32569, 48213, 60111, 58984, 44282, 55521, 16177, 12168], ...
_______________________________________________________________________________________________________________________
cat main.py  
from M import M  
  
flag = open("./flag.txt", "r").read().encode()  
  
n = 64  
assert len(flag) == n and flag[:4] == b"ctf{" and flag[-1:] == b"}"  
mod = 0x10001  
  
result = []  
  
assert len(M) == n  
for v in M:  
       assert len(v) == n  
  
for i in range(n):  
       dot = 0  
       for j in range(n):  
               dot += M[i][j] * flag[j]  
       result.append(dot % mod)  
  
print(result)  
# [29839, 662, 50523, 15906, 32667, 25159, 5172, 11685, 5618, 62174, 54405, 34902, 12259, 59526, 12299, 37286, 6055, 16813, 42488, 40708, 7662, 24263, 24047, 55429, 64420, 18167, 36330, 18325, 61471, 559, 32085, 23807, 26543, 26886, 24249, 45980, 23360, 15196, 42894, 33054, 22073, 23786, 63308, 44883, 60088, 38633, 54798, 42893, 29049, 25567, 33563, 49913, 63714, 51666, 60112, 19656, 13133, 11756, 34277, 55622, 14539, 54580, 48536, 1337]
```

The main things are
- M is a 64x64 matrix of integers
- result is a vector of length 64

To decrypt, compute the inverse matrix modulo 65537 and multiply it by the result vector, giving back the original flag bytes.

Solve script:
```python
import numpy as np

# Given data
M = [
    [57121, 19227, 32536, 277, 59722, 59191, 63981, 65014, 21161, 58022, 20206, 57613, 62723, 57302, 43570, 52022, 39155, 8180, 53332, 26348, 10715, 35123, 51712, 31978, 42383, 1938, 5076, 57300, 60128, 8229, 39370, 48263, 15527, 46890, 21181, 37520, 12962, 43620, 58636, 61383, 9476, 15805, 9435, 12791, 65295, 39577, 49230, 60291, 32630, 35471, 51322, 53976, 30361, 14514, 14231, 27362, 32569, 48213, 60111, 58984, 44282, 55521, 16177, 12168],
    [53887, 13460, 11431, 22534, 36534, 260, 51202, 18086, 49562, 16777, 22778, 58038, 37645, 27978, 28677, 61385, 3307, 26640, 30828, 9907, 6267, 21787, 28565, 7842, 19057, 32791, 44202, 47904, 42392, 61088, 3837, 2052, 23059, 44009, 54803, 10134, 766, 29865, 1860, 48964, 13568, 18556, 26416, 27122, 59108, 8108, 19813, 57300, 55768, 23485, 34060, 38195, 53103, 21430, 56561, 24, 2002, 42529, 30571, 11601, 64895, 33125, 47633, 9534], ...]

result = [29839, 662, 50523, 15906, 32667, 25159, 5172, 11685, 5618, 62174, 54405, 34902, 12259, 59526, 12299, 37286, 6055, 16813, 42488, 40708, 7662, 24263, 24047, 55429, 64420, 18167, 36330, 18325, 61471, 559, 32085, 23807, 26543, 26886, 24249, 45980, 23360, 15196, 42894, 33054, 22073, 23786, 63308, 44883, 60088, 38633, 54798, 42893, 29049, 25567, 33563, 49913, 63714, 51666, 60112, 19656, 13133, 11756, 34277, 55622, 14539, 54580, 48536, 1337]

n = 64
mod = 65537

# convert to numpy arrays modulo mod
M_np = np.array(M, dtype=np.int64) % mod
result_np = np.array(result, dtype=np.int64) % mod

# find modular inverse of matrix using gaussian elimination mod prime
def mod_mat_inv(matrix, prime):
    n = matrix.shape[0]
    a = np.concatenate([matrix, np.identity(n, dtype=np.int64)], axis=1)
    a = a % prime
    
    for i in range(n):
        pivot = i
        while pivot < n and a[pivot, i] == 0:
            pivot += 1
        if pivot == n:
            raise ValueError("Matrix is singular modulo prime")
        a[[i, pivot]] = a[[pivot, i]]
        
        inv = pow(int(a[i, i]), prime-2, prime)
        a[i] = (a[i] * inv) % prime
        
        # eliminate column i from other rows
        for j in range(n):
            if j != i:
                factor = a[j, i]
                a[j] = (a[j] - factor * a[i]) % prime
    
    return a[:, n:]

# compute inverse
try:
    M_inv = mod_mat_inv(M_np, mod)
except ValueError as e:
    print(e)
    exit()

flag_ints = (M_inv @ result_np) % mod

flag_bytes = bytes(int(x) for x in flag_ints)
print("Flag:", flag_bytes.decode())
```

`ctf{s0lve_th3_equ4t10n5_t0_f1nd_fl4g_h3r3_w4s_easy_en0ugh_NO???}`