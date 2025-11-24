---
layout: post
title:  "Patriot 2025 Cyptography"
date:   2025-11-24 17:09:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /patriot-ctf-2025-cryptography/
---
* TOC
{:toc}

## Password Palooza

**Category**: Cryptography

**Challenge author**: DJ Strigel

**Description**: We were infiltrating a hacker's network to bring down their team, but we've run into a slight issue. We managed to find the manager's password hash, and through some OSINT efforts we discovered that their password was leaked in a popular password breach. Their password also has two random digits following whatever password was leaked. An example would be password13. Can you crack their password? The flag format will be `pctf{password}`. 

Hash: `3a52fc83037bd2cb81c5a04e49c048a2`

I know the password has 2 random digits at the end. I added a custom rule to `john` which will append `00-99` to every word in the list:
```makefile
[List.Rules:AppendDigits]
Az"[0-9][0-9]"
```

Then I run john with this rule and the rockyou wordlist:
```shell
/usr/sbin/john --wordlist=/usr/share/wordlists/rockyou.txt --rules=AppendDigits --format=raw-md5 hash.txt  
Using default input encoding: UTF-8  
Loaded 1 password hash (Raw-MD5 [MD5 256/256 AVX2 8x3])  
Warning: no OpenMP support for this hash type, consider --fork=2  
Press 'q' or Ctrl-C to abort, almost any other key for status  
mr.krabbs57      (?)        
1g 0:00:00:43 DONE (2025-11-22 01:59) 0.02307g/s 18883Kp/s 18883Kc/s 18883KC/s mrb12357..motorolapink57  
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably  
Session completed.
```

`pctf{mr.krabbs57}`

___

## Cipher from Hell

**Category**: Cryptography

**Challenge author**: Neil Sharma

**Description**: We've recovered an encrypted message from the [eighth circle of Hell](https://en.wikipedia.org/wiki/Malbolge#Crazy_operation). Can you recover the hidden flag?

I get the challenge files `encryptor.py` and `encrypted`:
```shell
file encrypted  
encrypted: data
```

`encryptor.py`:
```shell
import math, sys  
  
inp = input("Enter your flag... ").encode()  
  
s = int.from_bytes(inp)  
  
o = (  
       (6, 0, 7),  
       (8, 2, 1),  
       (5, 4, 3)  
)  
  
c = math.floor(math.log(s, 3))  
  
if not c % 2:  
       sys.stderr.write("Error: flag length needs to be even (hint: but in what base system?)!\n")  
       sys.exit(1)  
  
ss = 0  
  
while c > -1:  
       ss *= 9  
       ss += o[s//3**c][s%3]  
  
       s -= s//3**c*3**c  
       s //= 3  
       c -= 2  
  
open("encrypted", 'wb').write(ss.to_bytes(math.ceil(math.log(ss, 256)), byteorder='big'))
```
The script takes the integer `s` from flag bytes. It represents `s` in base 3 and processes it as the most significant digit and least significant digit as a pair. It maps the MSD, LSD pairs through the `o` table to get a base 9 digit. The final base 9 digits form the encrypted integer `ss`.

I got a link to [https://en.wikipedia.org/wiki/Malbolge#Crazy_operation](https://en.wikipedia.org/wiki/Malbolge#Crazy_operation). It shows Malbolge's crz table:
```shell
crz   0  1  2
   0  1  0  0
   1  1  0  2
   2  2  2  1
```

The challenge script uses a different table:
```shell
o = (
    (6, 0, 7),
    (8, 2, 1),
    (5, 4, 3)
)
```

Solve script:
```python
import math

with open("encrypted", "rb") as f:
    data = f.read()
ss = int.from_bytes(data)

# inverse mapping
inv_map = {
    0: (0, 1),
    1: (1, 2),
    2: (1, 1),
    3: (2, 2),
    4: (2, 1),
    5: (2, 0),
    6: (0, 0),
    7: (0, 2),
    8: (1, 0)
}

# base 9 digits of ss (least significant first)
digits = []
n = ss
while n > 0:
    digits.append(n % 9)
    n //= 9

# get base 3 digits list
base3_digits = []
for g in digits:
    left, right = inv_map[g]
    base3_digits = [left] + base3_digits + [right]

# convert base3_digits to integer
s = 0
for d in base3_digits:
    s = s * 3 + d

# convert s to bytes
byte_len = (s.bit_length() + 7) // 8
flag_bytes = s.to_bytes(byte_len, 'big')
print(flag_bytes)
```

Output:
```shell
python3 solve.py  
b'pctf{a_l3ss_cr4zy_tr1tw1s3_op3r4ti0n_f37d4b}'
```

`pctf{a_l3ss_cr4zy_tr1tw1s3_op3r4ti0n_f37d4b}`

___

## Matrix Reconstruction

**Category**: Cryptography

**Challenge author**: DJ Strigel

**Description**: Someone’s been messing with your secure communication channel… and they’ve left traces! You’ve intercepted a mysterious ciphertext and a series of leaked internal states from a rogue pseudorandom generator. It seems the generator is powered by a secret 32×32 matrix A and an unknown 32-bit vector B. Your mission: reverse-engineer the system! Use the leaked states to reconstruct the hidden matrix, uncover the XOR constant, and decrypt the message. Only then will the true flag reveal itself. Remember: the keystream bytes come from the lowest byte of each internal state. Pay attention to the details.

I get the challenge files:
```shell
cat README.txt  
You are given ciphertext and leaked keystream states.  
Model: S[n+1] = A*S[n] XOR B  over GF(2)  
Recover A from consecutive states.  
Then decrypt.

file cipher.txt  
cipher.txt: data

xxd cipher.txt  
00000000: 6caa d8c8 a408 48f2 0000 9de6 c734 4c7b  l.....H......4L{  
00000010: 9c03 f5be bca9 6219 3aef 4a84 ad39 7a99  ......b.:.J..9z.  
00000020: ff04 f4                                  ...

cat keystream_leak.txt  
2694419740  
2430555337  
3055882924  
228605358  
4055459295  
676741477  
1030306057  
1320993926  
2317712498  
3680836913  
1922319333  
1836782265  
1490734773  
218490631  
4065897775  
3125259028  
189241330  
1710684784  
2355890305  
95797196  
813001417  
1021781706  
3522243094  
1603928614  
1122416469  
4125638785  
2423341845  
3666529189  
61609182  
2391267942  
148130332  
4246509548  
3552866507  
1487751530  
1895017353  
3277260507  
4251037246  
22647618  
3958787364  
227107204
```

The leaked states already contain the exact keystream used for encryption. I can make a brute force script to test each alignment. Once the alignment matches the real start, XOR to get the flag.

Solve script:
```python
#!/usr/bin/env python3
from pathlib import Path

leaks = [
    2694419740, 2430555337, 3055882924, 228605358, 4055459295,
    676741477, 1030306057, 1320993926, 2317712498, 3680836913,
    1922319333, 1836782265, 1490734773, 218490631, 4065897775,
    3125259028, 189241330, 1710684784, 2355890305, 95797196,
    813001417, 1021781706, 3522243094, 1603928614, 1122416469,
    4125638785, 2423341845, 3666529189, 61609182, 2391267942,
    148130332, 4246509548, 3552866507, 1487751530, 1895017353,
    3277260507, 4251037246, 22647618, 3958787364, 227107204
]

# read ciphertext
cipher = Path("cipher.txt").read_bytes()

def looks_ok(pt):
    return sum(32 <= c < 127 or c in (10,13) for c in pt) / len(pt) > 0.85

for i in range(len(leaks)):
    s = leaks[i]
    ks = bytearray()
    st = s
    for _ in range(len(cipher)):
        ks.append(st & 0xff)
        if i < len(leaks)-1:
            st = leaks[i+1]
            i += 1
        else:
            break

    pt = bytes(a ^ b for a,b in zip(cipher, ks))
    if looks_ok(pt):
        print(f"leak at {i}")
        print(pt.decode(errors='ignore'))
        raise SystemExit

print("found nothing :(")
```

Output:
```shell
python3 solve.py  
leak at 35  
pctf{mAtr1x_r3construct?on_!s_fu4n}
```

`pctf{mAtr1x_r3construct?on_!s_fu4n}`

___
