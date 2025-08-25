---
layout: post
title:  "BrunnerCTF 2025 Cryptography Challenges"
date:   2025-08-24 21:07:10 -0400
author: robo.uzi
tags: [CTF, crypto]
---
* TOC
{:toc}

## Shaken, Not Stirred
**Category:** Crypto

**Difficulty:** Beginner  
**Author:** Nissen
{% raw %}
After all that cake, I think it's time to sit back, relax, and have a drink 🍸 _One vodka martini, please - shaken, not stirred of course!_ ...Aw man, I just saw the bartender add something strange, I just wanted a good old classic. Wonder if I can demix it and get it out 🤔

Im given two files to download. `encrypt.py` and `output.txt`:
```t
import random  
import sys  
import time  
  
### HELPER FUNCTIONS - IGNORE ###  
def shake():  
   time.sleep(0.5)  
   for i in range(15):  
       sys.stdout.write(f"    {' ' * (i % 3)}慎 Shaking 慎")  
       sys.stdout.flush()  
       time.sleep(0.1)  
       sys.stdout.write("\b" * 20)  
       sys.stdout.write(" " * 20)  
       sys.stdout.write("\b" * 20)  
  
def printer(s):  
   time.sleep(0.5)  
   print(s)  
  
### PROGRAM ###  
printer(" Welcome to the Martini MiXOR 3000! \n")  
  
ingredients = [  
   "𧻓 Vodka",  
   " Dry Vermouth",  
   "龎 Olive Brine",  
   "㮝 Olive",  
   "異 Toothpick",  
]  
  
printer("Adding ingredients to the shaker...")  
shaker = 0  
for ingredient in ingredients:  
   # Mix ingredients together  
   shaker ^= len(ingredient) * random.randrange(18)  
   printer(f"  {ingredient}")  
  
printer("   Secret ingredient\n")  
with open("flag.txt", "rb") as f:  
   secret = f.read().strip()  
drink = bytes([b ^ shaker for b in secret])  
  
# Shake well!  
shake()  
shake()  
shake()  
  
if all(32 < d < 127 for d in drink):  
   printer("Drink's ready! Shaken, not stirred:")  
   printer(f" {drink.decode()} ")  
else:  
   printer("𧻓 Oops! Shook your drink too hard and spilled it 𧻓")
```

```shell
cat output.txt  
 Welcome to the Martini MiXOR 3000!   
  
Adding ingredients to the shaker...  
 𧻓 Vodka  
  Dry Vermouth  
 龎 Olive Brine  
 㮝 Olive  
 異 Toothpick  
  Secret ingredient  
  
Drink's ready! Shaken, not stirred:  
 wg`{{pgna}&J{!x&2fJWg`{{&g;;;_!x&fJWg`{{&gh 
```

Since I have the encrypted output and the script I can make a python script to attempt all XOR keys from 0-255 and recover the flag:
```python
encrypted = b"wg`{{pgna}&J{!x&2fJWg`{{&g;;;_!x&fJWg`{{&gh"

for key in range(256):
    candidate = bytes([b ^ key for b in encrypted])
    try:
        text = candidate.decode()
        if text.startswith("brunner"):
            print(f"Key: {key}, Flag: {text}")
    except UnicodeDecodeError:
        continue
```
`Key: 21, Flag: brunner{th3_n4m3's_Brunn3r...J4m3s_Brunn3r}`
{% endraw %}
___

## Half-Baked
**Category:** Crypto

**Difficulty:** Easy  
**Author:** Nissen

Boss wanted our cake webshop secured with RSA and said I should remember to use some good primes.  Not sure what that means, but he specifically asked for 2, I guess to save money. Sounded easy enough, but now the whole system feels a bit half-baked 😕

Need to break the RSA. I get a file containing this:
```shell
n = 2999882211429630485883650302877390551374775896896788078868325571891218714007953558505041388044334470201821965796391409921668122818083570668568660678895962925314655342154580738160357641047430373917156721861167458749434940591017306495880180805391185380307427539761080193213111534709378234670214284858143824384128077373871882033779166821558334466322908873171079631967672353755842618738501413251304204009472  
e = 65537  
c = 406899880095774364291729342954053590589397159355690238625035627993181937179155345315119680672959072539867481892078815991872758149967716015787715641627573675995588117336214614607141418649060621601912927211427125930492034626696064268888134600578061035823593102305974307471288655933533166631878786592162718700742194241218161182091193661813824775250046054642533470046107935752737753871183553636510066553725
```

I check to see `n` is a power of 2:
```python
import math

n = 2999882211429630485883650302877390551374775896896788078868325571891218714007953558505041388044334470201821965796391409921668122818083570668568660678895962925314655342154580738160357641047430373917156721861167458749434940591017306495880180805391185380307427539761080193213111534709378234670214284858143824384128077373871882033779166821558334466322908873171079631967672353755842618738501413251304204009472

log2_n = math.log2(n)
print(f"log2(n) = {log2_n}")
print(f"Is n a power of 2? {log2_n.is_integer()}")
```

```shell
python3 solve.py  
log2(n) = 1337.0  
Is n a power of 2? True
```

When n = 2^k in RSA, Euler's totient function φ(n) = 2^(k-1). This is a known RSA vulnerability. Using a modulus that's a power of 2 completely breaks the security. I can use this with python to get the flag:
```python
from Crypto.Util.number import long_to_bytes, inverse
import math

n = 2999882211429630485883650302877390551374775896896788078868325571891218714007953558505041388044334470201821965796391409921668122818083570668568660678895962925314655342154580738160357641047430373917156721861167458749434940591017306495880180805391185380307427539761080193213111534709378234670214284858143824384128077373871882033779166821558334466322908873171079631967672353755842618738501413251304204009472
e = 65537
c = 406899880095774364291729342954053590589397159355690238625035627993181937179155345315119680672959072539867481892078815991872758149967716015787715641627573675995588117336214614607141418649060621601912927211427125930492034626696064268888134600578061035823593102305974307471288655933533166631878786592162718700742194241218161182091193661813824775250046054642533470046107935752737753871183553636510066553725

# check n is power of 2 just cause
k = math.log2(n)
if k.is_integer():
    k = int(k)
    print(f"n = 2^{k}")
    
    # For n = 2^k, phi(n) = 2^(k-1)
    phi = 2**(k-1)
    
    # Calculate private key
    d = inverse(e, phi)
    
    m = pow(c, d, n)
    flag = long_to_bytes(m)
    print(f"Flag: {flag}")
else:
    print("n is not a power of 2")
```

```shell
python3 solve.py  
n = 2^1337  
Flag: b'brunner{s1ngl3_pr1m3_1s_d0ubl3_tr0ubl3}'
```

___

## The Cryptographic Kitchen!
**Category:** Crypto

**Difficulty:** Easy  
**Author:** H4N5

Our brand new baker, ElGamal, has whipped up the most wonderful cheesecake you could ever imagine. Seriously, it's so good it might encrypt your taste buds. But... there's a problem. One mysterious ingredient is missing from the recipe! Can you crack the code and figure out what ElGamal forgot to mix in?
{% raw %}
The suspiciously scrambled recipe says this:
```
p  = 14912432766367177751
g  = 2784687438861268863
h  = 8201777436716393968
c1 = 12279519522290406516
c2 = 10734305369677133991
```
The flag is the recovered plaintext, wrapped in `brunner{}`.  
Example: If you found `c4rr0t5`, the flag should be submitted as `brunner{c4rr0t5}`.

This is ElGamal encryption. Keys are generated like this:
```
Private key: x
Public key:  h = g^x mod p
```

Encrypting a message:
```
Choose random ephemeral key k
c1 = g^k mod p
c2 = m * h^k mod p
```
c1, c2 is the ciphertext. m is the plaintext. k is the ephemeral key. 

Decrypting if you know x or k:
```
m = c2 * (c1^x)^(-1) mod p
m = c2 * (h^k)^(-1) mod p
```

I make this python script to recover the flag:
```python
from sympy.ntheory import discrete_log

p  = 14912432766367177751
g  = 2784687438861268863
h  = 8201777436716393968
c1 = 12279519522290406516
c2 = 10734305369677133991

# solve for ephemeral key k using discrete log
k = discrete_log(p, c1, g)
print("Found k =", k)

# decrypt message with modular inverse
m = c2 * pow(h, -k, p) % p
print("Plaintext number:", m)
hexadecimal = hex(m)[2:]
flag_text = bytes.fromhex(hexadecimal).decode()
print("Flag:", f"brunner{{{flag_text}}}")
```

```shell
python3 decrypt.py  
Found k = 319224381322976394  
Plaintext number: 108256065500018  
Flag: brunner{buTT3r}
```
{% endraw %}

___

