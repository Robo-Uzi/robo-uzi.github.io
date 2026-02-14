---
layout: post
title:  "Crypto Challenges"
date:   2026-02-14 16:38:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /0xfun-2026-crypto-challenges/
---
* TOC
{:toc}

## The Slot Whisperer

**Category**: crypto

**Author**: SwitchCaseAdvocate

**Description**: The oldest slot machine in the casino runs on predictable gears. Watch it spin, learn its rhythm, and predict what comes next. Suffering ends in 31 minutes. `nc chall.0xfun.org 54893`

I get the challenge file `slot.py`:
```python
#!/usr/bin/env python3

class SlotMachineLCG:
    def __init__(self, seed=None):
        self.M = 2147483647
        self.A = 48271
        self.C = 12345
        self.state = seed if seed is not None else 1

    def next(self):
        self.state = (self.A * self.state + self.C) % self.M
        return self.state

    def spin(self):
        return self.next() % 100

if __name__ == "__main__":
    print("Slot Machine LCG Test")
    print("=" * 40)

    lcg = SlotMachineLCG(seed=12345)

    print("First 10 spins:")
    for i in range(10):
        spin = lcg.spin()
        print(f"Spin {i+1:2d}: {spin:2d}")
```

I connect to the server and see this:
```shell
nc chall.0xfun.org 54893  
39  
11  
45  
6  
69  
49  
42  
46  
1  
44  
Predict the next 5 spins (space-separated):
```

Run the solve script with the numbers output from the server:
```python
M = 2147483647
A = 48271
C = 12345

observed = [39, 11, 45, 6, 69, 49, 42, 46, 1, 44]

def next_state(s):
    return (A * s + C) % M

print("Bruteforcing...")

for k in range(M // 100 + 1):
    # state1 must satisfy state1 % 100 == observed[0]
    state1 = observed[0] + 100 * k

    test = state1
    ok = True

    # Verify spins 2–10
    for i in range(1, len(observed)):
        test = next_state(test)
        if test % 100 != observed[i]:
            ok = False
            break

    if ok:
        print("Recovered internal state after spin 1:", state1)

        # At this point:
        # test == state after spin 10
        current = test

        next5 = []
        for _ in range(5):
            current = next_state(current)
            next5.append(current % 100)

        print("Next 5 spins:", next5)
        break
```
LCGs are deterministic and parameters are known. The 10 outputs from the server are enough info to reconstruct the internal state. 

Run solve script:
```shell
python3 solve2.py  
Bruteforcing...  
Recovered internal state after spin 1: 1750578339  
Next 5 spins: [99, 70, 39, 81, 6]
```

Send the output of the next 5 numbers to the server and get the flag:
```shell
nc chall.0xfun.org 54893  
39  
11  
45  
6  
69  
49  
42  
46  
1  
44  
Predict the next 5 spins (space-separated): 99 70 39 81 6  
JACKPOT! You've mastered the slot machine!  
0xfun{sl0t_wh1sp3r3r_lcg_cr4ck3d}
```

`0xfun{sl0t_wh1sp3r3r_lcg_cr4ck3d}`

___

## Leonine Misbegotten

**Category**: crypto

**Author**: x03e

**Description**: At the end of Castle Morne awaits a fearsome lion with a goofy tail. His attacks come fast, feel random, yet follow a careful rhythm.

I get the challenge files. `chall.py` is used to encrypt the flag:
```python
from base64 import b16encode, b32encode, b64encode, b85encode  
from hashlib import sha1  
from random import choice  
from secret import flag  
  
SCHEMES = [b16encode, b32encode, b64encode, b85encode]  
  
ROUNDS = 16  
current = flag.encode()  
for _ in range(ROUNDS):  
	checksum = sha1(current).digest() # this is to help you check the integrity of your decryption  
	current = choice(SCHEMES)(current)  
	current += checksum  
  
with open("output", "wb") as f:  
	f.write(current)
```

`output` contains the encrypted flag:
```shell
cat output  
GRDDINZVGEZECNRZGRCTMQZVGE2TQNKBGRDDINZSIQZTMNBVGUZTGNJTHAZTONJTGRDDINZVGA3TINJTGQ3TENRUIU2EGNBYGRDDINZSGMZTKNZZGQ3TENRUIQ3UENBQGRDDINZVGEZECNSBGRDDGQZUG43DKM2EGRDDINZTIIZTQMRTGQ3TINJTG42TONCGGRDDINZVGE3TSNZXGRCDIRJXIU3TMNRTGRDDINZSIQZTMNBVGQ3TINJUIY3TGMZVGRDDINZVGE3TSNZXGRBTOMRXIE3TANJSGRDDINZSGMZTKNZZGQ3TINRWIUZTINJSGRDDINZSIQZUKNZUGQ3TINRWIUZTONCDGRDDINRXIUZTANRWGQ3TIMRSIQ3TANZVGRDDINZVIEZUENRYGRDDINJSIE2DSN2DGRDDINZVGEZDMNRVGQ3TINBXIQZTCNBQGRDDINZSIQ3ECNJUGUZTGMRWGE2UKNRUGRDDINZSIQZTMNBVGQ3TINJXGE3TQNBYGRDDINZXIE2UMNJUGUYDMNRUGE2EENZVGRDDINZVIEZUENRXGUZTGNBWIQZTOMRVGRDDINZVGEZUKNRYGRCDIRJTIUZECNZWGRDDINZTIIZTQMRTGUZTOQJTGA3EKNSGGRDDINRXIUZTANCFGUYDMNZUG42ECNZXGRDDINZSGMZTEMRRGUZDMMJWIE2 ... etc
```

Because the script appends a sha1 checksum to the end of each round, we can easily use this to reverse the process. 

Solve script:
```python
from base64 import b16decode, b32decode, b64decode, b85decode
from hashlib import sha1

DECODERS = [
    b16decode,
    b32decode,
    b64decode,
    b85decode
]

with open("output", "rb") as f:
    data = f.read()

for round_num in range(16):
    print(f"Round {round_num+1}")

    checksum = data[-20:]
    encoded = data[:-20]

    found = False

    for decoder in DECODERS:
        try:
            decoded = decoder(encoded)
        except Exception:
            continue

        if sha1(decoded).digest() == checksum:
            print(f"Correct decoder: {decoder.__name__}")
            data = decoded
            found = True
            break

    if not found:
        print("No valid decoder found :(")
        break

print("\nFinal result:")
print(data.decode())
```

Solve script output:
```shell
python3 solve4.py  
Round 1  
Correct decoder: b32decode  
Round 2  
Correct decoder: b16decode  
Round 3  
Correct decoder: b85decode  
Round 4  
Correct decoder: b32decode  
Round 5  
Correct decoder: b85decode  
Round 6  
Correct decoder: b64decode  
Round 7  
Correct decoder: b32decode  
Round 8  
Correct decoder: b32decode  
Round 9  
Correct decoder: b16decode  
Round 10  
Correct decoder: b16decode  
Round 11  
Correct decoder: b85decode  
Round 12  
Correct decoder: b64decode  
Round 13  
Correct decoder: b85decode  
Round 14  
Correct decoder: b64decode  
Round 15  
Correct decoder: b16decode  
Round 16  
Correct decoder: b16decode  
  
Final result:  
0xfun{p33l1ng_l4y3rs_l1k3_an_0n10n}
```

`0xfun{p33l1ng_l4y3rs_l1k3_an_0n10n}`

___
