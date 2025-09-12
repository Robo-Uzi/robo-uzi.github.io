---
layout: post
title:  "curve-desert"
date:   2025-09-11 19:29:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /watctf-2025-curve-desert/
---

**Title:** curve-desert

**Category:** crypto

 **Author:** [virchau13](https://github.com/virchau13)

**Description:** Dare I suspect myself of seeing it? The oasis of random entropy, just over the horizon?

`nc challs.watctf.org 3788`

I get this python script:
```shell
#!/usr/local/bin/python  
import ecdsa, random, os  
from Crypto.Util.number import bytes_to_long  
curve = ecdsa.curves.BRAINPOOLP512r1  
gen = curve.generator  
n = curve.order  
  
priv = random.randint(1, n-1)  
pub = priv * gen  
k = random.randint(1, n-1)  

challenge = os.urandom(32)  
print('Challenge hex:', challenge.hex())  
  
def sign(msg):  
   if msg == challenge:  
       print('Try harder than that!')  
       exit(1)  
   z = bytes_to_long(msg)  
   rpoint = k*gen  
   r = rpoint.x() % n  
   assert r != 0  
   s = (pow(k, -1, n) * (z + r*priv)) % n  
   return (int(r), int(s))  
  
def verify(msg, r, s):  
   z = bytes_to_long(msg)  
   u1 = (pow(s, -1, n) * z) % n  
   u2 = (pow(s, -1, n) * r) % n  
   rpoint = u1*gen + u2*pub  
   return rpoint.x() % n == r  
  
assert verify(b'hello', *sign(b'hello'))  
  
def menu():  
   print('Menu options:')  
   print('[1] Sign')  
   print('[2] Verify')  
   choice = int(input('Choose an option: ').strip())  
   if choice == 1:  
       msghex = input('Input hex of message to sign: ').strip()  
       r, s = sign(bytes.fromhex(msghex))  
       print(f'Your signature is: {r} {s}')  
   elif choice == 2:  
       msghex = input('Input hex of message to verify: ').strip()  
       line = input('Input the two integers of the signature seperated by a space: ').strip()  
       r, s = [int(x) for x in line.split(' ')]  
       msg = bytes.fromhex(msghex)  
       if verify(msg, r, s):  
           print('Message verified successfully!')  
           if msg == challenge:  
               print('You have passed the challenge! Your reward:')  
               print(open('flag.txt', 'r').read())  
       else:  
           print('Invalid signature.')  
  
while True:  
   menu()
```

Connecting remotely I see this:
```shell
nc challs.watctf.org 3788  
Challenge hex: d175e6622e349bb6a472e2346a761902ea18adf1773201332a1ba6c4e561b28e  
Menu options:  
[1] Sign  
[2] Verify  
Choose an option: 1  
Input hex of message to sign: AAAA  
Your signature is: 3296764691325409228700025337846500047098589246381104162579896628221385203068430638807536599116944572246543953545693958347817887747740141323640132156851036 347799530612934086932743550258911352472665258106696845113606266456636223398537776387007049304741049773059716736296833873698769114294389414368705035506039  
Menu options:  
[1] Sign  
[2] Verify  
Choose an option: 
```

The script uses ECDSA (Elliptic Curve Digital Signature Algorithm) on the BrainpoolP512r1 curve. It creates a random private key (`priv`) and (`k`) which is a number that should only be used once. When connecting you get the challenge hex. You can sign other messages and verify signatures. 

`k` is fixed and reused for all messages. Using two signatures on different messages, you can compute `k` (`k = s1​−s2​ / z1​−z2 ​​mod n`) and then the private key (`priv = s*k−z / r ​mod n`). Once you have the private key you can sign the challenge and get the flag. 

Run this script:
```python
from pwn import remote
from Crypto.Util.number import bytes_to_long
import ecdsa

curve = ecdsa.curves.BRAINPOOLP512r1
n = curve.order

p = remote("challs.watctf.org", 3788)
challenge = bytes.fromhex(p.recvline_contains(b"Challenge hex:").decode().split()[-1])
chal = bytes_to_long(challenge)

# request a signature
def sig(msg: bytes):
    z = bytes_to_long(msg)
    p.sendlineafter(b"Choose an option:", b"1")
    p.sendlineafter(b"Input hex of message to sign:", msg.hex().encode())
    r, s = map(int, p.recvline().decode().split()[-2:])
    return z, r, s

# get two signatures
z1, r, s1 = sig(b"AAAA")
z2, _, s2 = sig(b"BBBB")

# recover k and private key
k = ((z1 - z2) * pow(s1 - s2, -1, n)) % n
priv = ((s1 * k - z1) * pow(r, -1, n)) % n

sign = (pow(k, -1, n) * (chal + r * priv)) % n

p.sendlineafter(b"Choose an option:", b"2")
p.sendlineafter(b"Input hex of message to verify:", challenge.hex().encode())
p.sendlineafter(b"Input the two integers of the signature seperated by a space:", f"{r} {sign}".encode())
print(p.recvuntil(b"}").decode())
```
`watctf{yeah_dont_share_the_k_parameter_it_doesnt_work_out}`