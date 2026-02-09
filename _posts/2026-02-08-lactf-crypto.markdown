---
layout: post
title:  "Crypto Challenges"
date:   2026-02-08 23:29:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /lactf-2026-crypto-challenges/
---
* TOC
{:toc}

## smol cats

**Category**: crypto

**Author**: burturt

**Description**: My cat walked across my keyboard and made this RSA implementation, encrypting the location of the treats they stole from me! However, they already got fed twice today, and are already overweight and needs to lose some weight, so I cannot let them eat more treats. Can you defeat my cat's encryption so I can find their secret stash of treats and keep my cat from overeating? `nc chall.lac.tf 31224`

I connect to the server and see this:
```shell
nc chall.lac.tf 31224  
 /\_/\     
( o.o )    
 > ^ <     
  
*meow* Welcome to my cat cafe!  
I'm a hungry kitty and I've hidden my treats in a secret place.  
I will let you know where I hid them if you can defeat my encryption >.<  
I encrypted the number of treats I want with RSA... but my paws are small,  
so I used tiny primes. *purrrrr*  
  
n = 1127628179438722433420960193106064960970090935949289903664879  
e = 65537  
c = 260506153769234577226670390044781857239536987376407549213642  
  
*mrrrow?* How many treats do I want?
```

I dont have sage installed so I go to [https://sagecell.sagemath.org/](https://sagecell.sagemath.org/) and run `factor(1127628179438722433420960193106064960970090935949289903664879)` and it factorizes into `908899685173810100515431357911 * 1240651963943726739066076062889`.

Recover the ciphertext with this python script:
```python
n = 1127628179438722433420960193106064960970090935949289903664879
e = 65537
c = 260506153769234577226670390044781857239536987376407549213642
p = 908899685173810100515431357911
q = 1240651963943726739066076062889

phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)
m = pow(c, d, n)
print(m)
```

I send the output to the server to get the flag:
```shell
nc chall.lac.tf 31224  
 /\_/\     
( o.o )    
 > ^ <     
  
*meow* Welcome to my cat cafe!  
I'm a hungry kitty and I've hidden my treats in a secret place.  
I will let you know where I hid them if you can defeat my encryption >.<  
I encrypted the number of treats I want with RSA... but my paws are small,  
so I used tiny primes. *purrrrr*  
  
n = 1127628179438722433420960193106064960970090935949289903664879  
e = 65537  
c = 260506153769234577226670390044781857239536987376407549213642  
  
*mrrrow?* How many treats do I want? 285396583453059016452447020608155246038062146651185259098886  
*PURRRRRR* You got it right! You may pet me now.  
Here's your reward, human:  
lactf{sm0l_pr1m3s_4r3_n0t_s3cur3}
```

`lactf{sm0l_pr1m3s_4r3_n0t_s3cur3}`

___

## the-clock

**Category**: crypto

**Author**: freed

**Description**: Don't run out of time

I get two challenge files:
```shell
cat output.txt  
Alice's public key:  (13109366899209289301676180036151662757744653412475893615415990437597518621948, 52147230114829273649400193055104479862837573645083769594969383745041757478  
1)  
Bob's public key:  (1970812974353385315040605739189121087177682987805959975185933521200533840941, 12973039444480670818762166333866292061530850590498312261363790018126209960024  
)  
Encrypted flag:  d345a465538e3babd495cd89b43a224ac93614e987dfb4a6d3196e2d0b3b57d9
```

And `chall.py`:
```python
from random import randrange  
from Crypto.Cipher import AES  
from Crypto.Util.Padding import pad  
from hashlib import md5  
p =  
def clockadd(P1,P2):  
    x1,y1 = P1  
    x2,y2 = P2  
    x3 = x1*y2+y1*x2  
    y3 = y1*y2-x1*x2  
    return x3,y3  
  
def F(p):  
    # caveat: caller must ensure that p is prime  
    class F:  
    def __init__(self,x):  
        self.int = x % p  
    def __str__(self):  
        return str(self.int)  
    __repr__ = __str__  
    def __eq__(a,b):  
        return a.int == b.int  
    def __ne__(a,b):  
        return a.int != b.int  
    def __add__(a,b):  
        return F(a.int + b.int)  
    def __sub__(a,b):  
        return F(a.int - b.int)  
    def __mul__(a,b):  
        return F(a.int * b.int)  
    def __div__(a,b):  
        # caveat: caller must ensure that b is nonzero  
        return a*F(pow(b.int,p-2,p))  
    return F  
  
def scalarmult(P, n):  
    # caveat: caller must ensure that n is nonnegative  
    # caveat: n is limited by python's recursion limit  
    if n == 0: return (Fp(0),Fp(1))  
    if n == 1: return P  
    Q = scalarmult(P, n//2)  
    Q = clockadd(Q,Q)  
    if n % 2: Q = clockadd(P,Q)  
    return Q  
  
Fp = F(p)  
  
base_point = (Fp(13187661168110324954294058945757101408527953727379258599969622948218380874617),Fp(5650730937120921351586377003219139165467571376033493483369229779706160055207  
))  
  
alice_secret = randrange(2**256)  
alice_public = scalarmult(base_point, alice_secret)  
print("Alice's public key: ", alice_public)  
bob_secret = randrange(2**256)  
bob_public = scalarmult(base_point, bob_secret)  
print("Bob's public key: ", bob_public)  
  
assert scalarmult(bob_public, alice_secret) == scalarmult(alice_public, bob_secret)  
shared_secret = scalarmult(bob_public, alice_secret)  
key = md5(f"{shared_secret[0].int},{shared_secret[1].int}".encode()).digest()  
FLAG = b""  
print("Encrypted flag: ", AES.new(key, AES.MODE_ECB).encrypt(pad(FLAG, 16)).hex())
```

On [https://sagecell.sagemath.org/](https://sagecell.sagemath.org/) I run this:
```sage
p = 13767529254441196841515381394007440393432406281042568706344277693298736356611
q = p + 1
factor(q)
```

Output:
```shell
2^2 * 39623 * 41849 * 42773 * 46511 * 47951 * 50587 * 50741 * 51971 * 54983 * 55511 * 56377 * 58733 * 61843 * 63391 * 63839 * 64489
```

Solve script:
```python
from math import gcd
from sympy.ntheory.modular import crt
from hashlib import md5
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

gx = 13187661168110324954294058945757101408527953727379258599969622948218380874617
gy = 5650730937120921351586377003219139165467571376033493483369229779706160055207

ax = 13109366899209289301676180036151662757744653412475893615415990437597518621948
ay = 5214723011482927364940019305510447986283757364508376959496938374504175747801

bx = 1970812974353385315040605739189121087177682987805959975185933521200533840941
by = 12973039444480670818762166333866292061530850590498312261363790018126209960024

ct = bytes.fromhex(
    "d345a465538e3babd495cd89b43a224a"
    "c93614e987dfb4a6d3196e2d0b3b57d9"
)

def norm(x, y):
    return x*x + y*y - 1

p = gcd(norm(gx, gy), gcd(norm(ax, ay), norm(bx, by)))
order = p + 1

def mul(P, Q):
    x1,y1 = P
    x2,y2 = Q
    return (
        (x1*y2 + y1*x2) % p,
        (y1*y2 - x1*x2) % p
    )

def pow_point(P, n):
    R = (0,1)
    while n:
        if n & 1:
            R = mul(R, P)
        P = mul(P, P)
        n >>= 1
    return R

# factors from sage
factors = [
    (2,2),
    (39623,1),(41849,1),(42773,1),(46511,1),(47951,1),
    (50587,1),(50741,1),(51971,1),(54983,1),(55511,1),
    (56377,1),(58733,1),(61843,1),(63391,1),(63839,1),(64489,1)
]

G = (gx, gy)
A = (ax, ay)

residues = []
moduli = []

for prime, exp in factors:
    pe = prime**exp
    m = order // pe

    Gm = pow_point(G, m)
    Am = pow_point(A, m)

    cur = (0,1)
    for k in range(pe):
        if cur == Am:
            residues.append(k)
            moduli.append(pe)
            break
        cur = mul(cur, Gm)

sol, mod = crt(moduli, residues)
assert sol is not None
a = int(sol)

B = (bx, by)
shared = pow_point(B, a)

key = md5(f"{shared[0]},{shared[1]}".encode()).digest()
flag = unpad(AES.new(key, AES.MODE_ECB).decrypt(ct), 16)

print(flag.decode())
```

I am not very good at crypto so...... here is the AI's explanation for breaking this encryption:
```
The challenge implements Diffie–Hellman over the group of norm-1 elements in 𝔽ₚ², using complex-number-like multiplication.

The modulus `p` is not provided, but can be recovered by taking the GCD of `x² + y² − 1` across public points.

The group order is `p + 1`, which factors completely into small primes. This allows a Pohlig–Hellman attack to recover Alice’s private key by solving discrete logs modulo each factor and recombining with CRT.

Once the private key is known, the shared secret is computed normally and the AES key derived via MD5, allowing decryption of the flag.
```

`lactf{t1m3_c0m3s_f4r_u_4all}`

___
