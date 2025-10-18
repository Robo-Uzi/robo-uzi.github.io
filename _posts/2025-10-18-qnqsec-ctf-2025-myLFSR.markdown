---
layout: post
title:  "myLFSR?"
date:   2025-10-18 15:47:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-myLFSR/
---

**Title:** myLFSR?

**Category:** crypto

**Description:** A custom LFSR working in GF(3). Computing is easy, predicting is hard. As for getting the flag? That's another story.

**Author:** Swizzer

I download the challenge files. Contants of `chall.py`:
```python
from secrets import randbits  
import os  
  
  
def expand(n: int, base=3) -> list[int]:  
    res = []  
    while n:  
    res.append(n % base)  
    n //= base  
return res  
  
  
class myLFSR:  
    def __init__(self, key: list[int], mask: list[int]):  
        assert all(0 <= x < 3 for x in key), "Key must be in range [0, 2]"  
        assert all(0 <= x < 3 for x in mask), "Mask must be in range [0, 2]"  
        assert len(key) == len(mask), "Key and mask must be of the same length"  
  
        self.state = key  
        self.mask = mask  
        self.mod = 3  
  
def __call__(self) -> int:  
    b = sum(s * m for s, m in zip(self.state, self.mask)) % self.mod  
    output = self.state[0]  
    self.state = self.state[1:] + [b]  
  
return output  
  
  
class Cipher:  
    def __init__(self, key: list[int], mask: list[int]):  
        self.lfsr = myLFSR(key, mask)  
  
def encrypt(self, msg: bytes) -> bytes:  
    pt = expand(int.from_bytes(msg, "big"))  
    stream = [self.lfsr() for _ in range(len(pt))]  
    ct = [a ^ b for a, b in zip(pt, stream)]  
return bytes(ct)  
  
  
if __name__ == "__main__":  
    flag = open("flag.txt", "rb").read().strip()  
    KEY = expand(int.from_bytes(os.urandom(8), "big"))  
    MASK = [randbits(256) % 3 for _ in range(len(KEY))]  
    cipher = Cipher(KEY, MASK)  
    gift = cipher.encrypt(b"\xff" * (len(KEY) // 3 + 3))  
    ct = cipher.encrypt(flag)  
    print(len(KEY))  
    print(gift.hex())  
    print(ct.hex())
```

Contents of `output.txt`:
```shell
40  
020000030303020100010000020302000001000100000301030302000001010301010300000002000301030300000200000301020100030302000000020100000001030100010300020203010103000102
01010002000003000002010000000203030201000000030100030003010103030001020200010301000202000202030302020103030002030100030002020000010200010000010100000200000103010103030302010301010100000000030000020203030300020103010002020100020201000100000000030003010301010101000101010002000203000102020002000000000102030303000000000301010102030002030100010002000203000000010200000000030001030003020303000300000302010303000301020302000003020003030102010102020000030201000103000100
```
output.txt contains three lines:
1) key length (int)
2) hex of gift ciphertext
3) hex of flag ciphertext

The solve script will join the lines after the length line and split them into gift/flag ciphertext based on the expected gift length in bytes.

Solve script:
```python
#!/usr/bin/env python3
from typing import List, Optional
from math import ceil

def read_output(path: str = "output.txt"):
    with open(path, "r") as f:
        lines = [ln.strip() for ln in f if ln.strip() != ""]
    if len(lines) < 3:
        raise SystemExit("output.txt must contain at least 3 non empty lines (n, gift_hex, ct_hex).")
    n = int(lines[0])
    # Join the rest and try to split into two hex strings (gift then ct)
    rest = "".join(lines[1:])
    return n, rest

def base3_expand_from_bytes(msg_bytes: bytes) -> List[int]:
    """Expand integer represented by msg_bytes into little endian base 3 digits."""
    x = int.from_bytes(msg_bytes, "big")
    if x == 0:
        return [0]
    digs = []
    while x:
        digs.append(x % 3)
        x //= 3
    return digs

def gauss_mod3(A: List[List[int]], b: List[int]) -> Optional[List[int]]:
    """Solve A x = b over Z_3. If multiple solutions, this returns one with free vars = 0.
       Returns None if inconsistent."""
    rows = len(A)
    cols = len(A[0])
    # build augmented matrix, reduce mod3
    M = [ [(A[r][c] % 3) for c in range(cols)] + [b[r] % 3] for r in range(rows) ]
    r = 0
    pivots = []
    for c in range(cols):
        if r >= rows: break
        # find row with nonzero in col c
        sel = None
        for i in range(r, rows):
            if M[i][c] % 3 != 0:
                sel = i; break
        if sel is None:
            continue
        M[r], M[sel] = M[sel], M[r]
        pivots.append(c)
        # normalize pivot to 1 (inverse of 2 is 2 in mod3)
        pv = M[r][c] % 3
        if pv == 2:
            for j in range(c, cols+1):
                M[r][j] = (M[r][j] * 2) % 3
        # eliminate other rows
        for i in range(rows):
            if i == r: continue
            factor = M[i][c] % 3
            if factor != 0:
                for j in range(c, cols+1):
                    M[i][j] = (M[i][j] - factor * M[r][j]) % 3
        r += 1
    # check inconsistency
    for i in range(r, rows):
        if all(M[i][j] == 0 for j in range(cols)) and M[i][cols] != 0:
            return None
    # back fill solution (free vars = 0)
    x = [0] * cols
    for i_row, c in enumerate(pivots):
        x[c] = M[i_row][cols] % 3
    return x

def lfsr_step_and_stream(state: List[int], mask: List[int], steps: int):
    """Return (outputs_list, final_state). Outputs are the state[0] values produced each step."""
    st = list(state)
    out = []
    for _ in range(steps):
        b = sum(s*m for s,m in zip(st, mask)) % 3
        out.append(st[0])
        st = st[1:] + [b]
    return out, st

def main():
    n, rest_hex = read_output("output.txt")

    # gift plaintext length in bytes used in challenge: (n // 3 + 3)
    gift_msg_bytes = (n // 3) + 3

    # compute base 3 expansion length of the gift plaintext integer
    pt_gift = base3_expand_from_bytes(b"\xff" * gift_msg_bytes)
    pt_gift_len = len(pt_gift)

    # split the concatenated hex into gift_hex (pt_gift_len bytes) and ct_hex (rest)
    if len(rest_hex) < pt_gift_len * 2:
        raise SystemExit("Not enough hex data to extract gift. Check output.txt formatting.")
    gift_hex = rest_hex[: pt_gift_len * 2]
    ct_hex = rest_hex[pt_gift_len * 2 : ]

    gift_bytes = bytes.fromhex(gift_hex)
    ct_bytes = bytes.fromhex(ct_hex)

    # Recover stream outputs from gift: stream = pt_gift ^ gift_bytes
    # If some outputs are 3 (because of xor), reduce mod 3 to get values in {0,1,2}.
    stream_known = [(a ^ b) % 3 for a, b in zip(pt_gift, gift_bytes)]

    if len(stream_known) <= n:
        raise SystemExit("Not enough recovered outputs to solve for mask; need > n outputs.")

    # Build linear equations: for t in [0 .. m-n-1]: sum mask[i]*y_{t+i} = y_{t+n}  (mod 3)
    m = len(stream_known)
    eqs = []
    rhs = []
    for t in range(m - n):
        eqs.append([stream_known[t + i] for i in range(n)])
        rhs.append(stream_known[t + n])

    mask = gauss_mod3(eqs, rhs)
    if mask is None:
        raise SystemExit("Failed to solve linear system for mask (inconsistent).")

    # initial state (first n outputs)
    initial_state = stream_known[:n]

    # advance the LFSR by the length of the gift plaintext digits (we already consumed these outputs)
    _, state_after_gift = lfsr_step_and_stream(initial_state, mask, pt_gift_len)

    # produce keystream for the flag length (ct_bytes length)
    ks_flag, _ = lfsr_step_and_stream(state_after_gift, mask, len(ct_bytes))

    # recover base 3 digits of flag plaintext as ks ^ ct_byte (then modulo 3)
    pt_flag_digits = [ (ks ^ cb) % 3 for ks, cb in zip(ks_flag, ct_bytes) ]

    # convert little endian base 3 digits to integer then to bytes
    val = 0
    mul = 1
    for d in pt_flag_digits:
        val += d * mul
        mul *= 3

    if val == 0:
        print("Recovered integer is 0 (empty flag?).")
        return

    flag_bytes = val.to_bytes((val.bit_length() + 7) // 8, "big")

    try:
        print("flag:", flag_bytes.decode())
    except UnicodeDecodeError:
        print("flag (hex):", flag_bytes.hex())

if __name__ == "__main__":
    main()
```
I have never worked with linear feedback shift registers so I used AI to generate this script. 

From what I understand the challenge produces a keystream that is XORed with the message to make the ciphertext. The `gift` is ciphertext of a known plaintext. This is important because it reveals part of the keystream.

First XOR the known plaintext (`gift`) with its ciphertext. These outputs are numbers 0, 1, or 2. Then using the known outputs, we can set up a system of equations and solve it mod 3 (since the LFSR only uses numbers 0,1,2). After solving I know the mask which this tells me how the LFSR generates its numbers.

Finally the script simulates the LFSR from the initial state using the mask, and skips the number of steps used for the gift. Continue producing outputs for the length of the flag ciphertext. XOR the keystream with the flag ciphertext which gives the base 3 digits of the flag. Convert those digits back into bytes and recover the flag.

`QnQSec{i_L1K3_B3RleK4mP_m4Ss3y_0n_m0d_3_f1elD}`