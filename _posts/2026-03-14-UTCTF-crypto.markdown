---
layout: post
title:  "Crypto Challenges"
date:   2026-03-09 17:43:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UTCTF-2026-crypto/
---
* TOC
{:toc}

## Fortune Teller

**Category**: cryptography

**Description**: Our security team built a "cryptographically secure" random number generator. The lead engineer assured us it was basically AES. He has since been let go. By Garv (@GarvK07 on discord)

I get the challenge file:
```shell
cat lcg.txt  
We're using a Linear Congruential Generator (LCG) defined as:  
  
  x_(n+1) = (a * x_n + c) % m  
  
where m = 4294967296 (2^32), and a and c are secret.  
  
We intercepted the first 4 outputs of the generator:  
  
  output_1 = 4176616824  
  output_2 = 2681459949  
  output_3 = 1541137174  
  output_4 = 3272915523  
  
The flag was encrypted by XORing it with output_5 (used as a 4-byte repeating key).  
  
  ciphertext (hex) = 3cff226828ec3f743bb820352aff1b7021b81b623cff31767ad428672ef6
```

The challenge uses a LCG defined by `x_(n+1)​=(a*x_n​+c) mod m` where `m=2^32`. Because four consecutive outputs of the generator are given, we can recover the secret parameters `a` and `c`. By subtracting two consecutive equations, we eliminate `c` and solve for `a` using modular arithmetic and a modular inverse. Once `a` is known we substitute it back into the generator equation to compute `c`. With both parameters recovered, we can predict the next output `x5`​. The challenge states that `x5` is used as a 4 byte repeating XOR key to encrypt the flag so we convert `x5` to four bytes and XOR it repeatedly with the ciphertext to recover the plaintext. This works because LCGs are not cryptographically secure. If an attacker observes a few outputs they can reconstruct the generator and predict future values.

Solve script:
```python
from Crypto.Util.number import inverse

m = 2**32

x1 = 4176616824
x2 = 2681459949
x3 = 1541137174
x4 = 3272915523

# recover a
a = ((x3 - x2) * inverse(x2 - x1, m)) % m

# recover c
c = (x2 - a * x1) % m

# predict next output
x5 = (a * x4 + c) % m

print("a =", a)
print("c =", c)
print("x5 =", x5)

cipher = bytes.fromhex("3cff226828ec3f743bb820352aff1b7021b81b623cff31767ad428672ef6")

key = x5.to_bytes(4,'big')

flag = bytes([cipher[i] ^ key[i % 4] for i in range(len(cipher))])

print(flag)
```

Output:
```shell
python3 solve4.py  
a = 3355924837  
c = 2915531925  
x5 = 1233863684  
b'utflag{pr3d1ct_th3_futur3_lcg}'
```

`utflag{pr3d1ct_th3_futur3_lcg}`

___

## Smooth Criminal

**Category**: cryptography

**Description**: Our cryptographer assured us that a 649-bit prime makes this completely unbreakable. He also said the order of the group "doesn't really matter that much." He no longer works here. By Garv (@GarvK07 on discord)

I get the challenge file:
```shell
cat dlp.txt  
The flag has been encoded as a secret exponent x, where:  
  
h = g^x mod p  
  
Your job: find x. Convert it from integer to bytes to get the flag.  
  
p = 1363402168895933073124331075716158793413739602475544713040662303260999503992311247861095036060712607168809958344896622485452229880797791800555191761456659256252204001928525518751268009081850267001  
g = 223  
h = 1009660566883490917987475170194560289062628664411983200474597006489640893063715494610197294704009188265361176318190659133132869144519884282668828418392494875096149757008157476595873791868761173517
```

This is a discrete logarithm problem `h=g^x mod p` where you must recover the secret exponent `x` which encodes the flag. When the group order is smooth, the Pohlig-Hellman algorithm can efficiently reduce the discrete logarithm problem into several much smaller discrete logs modulo each prime power factor of `p−1`. The smaller solutions are then recombined using the chinese remainder theorem to recover the full exponent `x`. The `sympy.ntheory discrete_log` function automatically detects this structure and applies the algorithms internally allowing the exponent to be recovered quickly despite the large prime. Once `x` is found it is converted from an integer to bytes revealing the flag.

Solve script:
```python
from sympy.ntheory import discrete_log
from Crypto.Util.number import long_to_bytes

p = 1363402168895933073124331075716158793413739602475544713040662303260999503992311247861095036060712607168809958344896622485452229880797791800555191761456659256252204001928525518751268009081850267001
g = 223
h = 1009660566883490917987475170194560289062628664411983200474597006489640893063715494610197294704009188265361176318190659133132869144519884282668828418392494875096149757008157476595873791868761173517

x = discrete_log(p, h, g)
print(long_to_bytes(x))
```

Output:
```shell
python3 solve5.py  
b'utflag{sm00th_cr1m1nal_caught}'
```

`utflag{sm00th_cr1m1nal_caught}`

___

## Oblivious Error

**Category**: cryptography

**Description**: My friend made an RSA-based 1-2 oblivious transfer protocol program. I don't know what that means but I need to know quick because I accidentally deleted his code! I replaced the part I deleted with the following code in the text file below, but now one of the messages is undecodable and I don't know why! Can you decode the lost message? By Joel (@jayscx on discord) `nc challenge.utctf.live 8379`

I get the challenge file `my-code.txt`:
```shell
while True:
    try:
        print("Please pick a value k.")
        k = int(input())
        break
    except ValueError:
        print("Invalid value. Please pick an integer.")
        print("Please pick a value k.")
        k = int(input())

v = (x0 + (int(k) ^ e)) % N
```
The challenge implements an oblivious transfer (OT) using RSA. The code uses XOR instead of modular exponentiation for RSA. So the server computes: `k_i = (v - x_i)^d % N` for `i = 0, 1`

When I connect to the server I see this:
```shell
nc challenge.utctf.live 8379  
Here's your entrance key!  
N = 6369025063238790180722140998639295588664035173147548735537594312433024540341405264126417120138921699106671025829261506642881343969178511286348922327302733  
e = 65537  
Here are my random x values!  
x0: 3656800175596208423017798444979883719052863014747677254711689537262202425331181796477811013092477681100914615589864545471170639068500120076065186765531027  
x1: 6082239663751533050749806404767247363551590901054887291597876473525817775291404817890387576384590702953346173013735923691221939358026385086031940638665231  
Please pick a value k.  
2  
Here are your messages!  
Message 1: 2144427350118966923826844658243266929661788607835795743446543738888009139466495740799492973614190586532888765121964782048247170612665830796045450000502742  
Message 2: 4445058237281566285088142925852042484181481649337791489081002014987003068646209269361858001815669715668910199057289640995190447414197577559611607763261936
```

Solve script:
```python
#!/usr/bin/env python3
from pwn import *
from Crypto.Util.number import long_to_bytes

HOST = "challenge.utctf.live"
PORT = 8379

def solve():
    io = remote(HOST, PORT)
    io.recvuntil(b"N = ")
    N = int(io.recvline())
    io.recvuntil(b"e = ")
    e = int(io.recvline())
    io.recvuntil(b"x0: ")
    x0 = int(io.recvline())
    io.recvuntil(b"x1: ")
    x1 = int(io.recvline())

    # make k1_server = 1 so m1 = M2 - 1
    # need: k XOR e = (x1 - x0 + 1) % N
    target = (x1 - x0 + 1) % N
    k = target ^ e

    io.recvuntil(b"value k.")
    io.sendline(str(k).encode())
    io.recvuntil(b"Message 1:")
    M1 = int(io.recvline())
    io.recvuntil(b"Message 2:")
    M2 = int(io.recvline())
    io.close()

    m1 = (M2 - 1) % N
    print("Message 1 bytes:", long_to_bytes(m1))
    try:
        print("Decoded:", long_to_bytes(m1).decode())
    except Exception as ex:
        print("Decode error:", ex)

solve()
```

Output:
```shell
python3 solve6.py  
[+] Opening connection to challenge.utctf.live on port 8379: Done  
[*] Closed connection to challenge.utctf.live port 8379  
Message 1 bytes: b'utflag{my_obl1v10u5_fr13nd_ru1n3d_my_c0de}\n'  
Decoded: utflag{my_obl1v10u5_fr13nd_ru1n3d_my_c0de}
```

`utflag{my_obl1v10u5_fr13nd_ru1n3d_my_c0de}`
