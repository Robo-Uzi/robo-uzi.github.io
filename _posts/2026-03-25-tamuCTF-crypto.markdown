---
layout: post
title:  "Crypto Challenges"
date:   2026-03-25 20:14:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /tamuCTF-2026-crypto/
---
* TOC
{:toc}

## Hidden Log Factoring

**Category**: crypto

**Author**: `Marfung37`

**Description**: Where are the logs? I got the raw data in `data.txt` but can't find the logs.

I get the challenge files:
```shell
cat requirements.txt  
pycryptodome  
cryptography  

cat data.txt  
---- RSA Public Data ----  
n=71016310005824589926747341243598522145452505235842335510488353587223142066921470760443852767377534776713566052988373656012584808377496091765373981120165220471527586994259252074709653090148780742972203779666231432769553199154214563039426087870098774883375566546770723222752131892953195949848583409407713489831  
e=65537  
---- DLP Public Data ----  
p=200167626629249973590210748210664315551571227173732968065685194568612605520816305417784745648399324178485097581867501503778073506528170960879344249321872139638179291829086442429009723480288604047975360660822750743411854623254328369265079475034447044479229192540942687284442586906047953374527204596869578972378578818243592790149118451253249  
g=11  
A=44209577951808382329528773174800640982676772266062718570752782238450958062000992024007390942331777802579750741643234627722057238001117859851305258592175283446986950906322475842276682130684406699583969531658154117541036033175624316123630171940523312498410797292015306505441358652764718889371372744612329404629522344917215516711582956706994  
---- Encrypted Data ----  
D=9478993126102369804166465392238441359765254122557022102787395039760473484373917895152043164556897759129379257347258713397227019255397523784552330568551257950882564054224108445256766524125007082113207841784651721510041313068567959041923601780557243220011462176445589034556139643023098611601440872439110251624  
c=1479919887254219636530919475050983663848182436330538045427636138917562865693442211774911655964940989306960131568709021476461747472930022641984797332621318327273825157712858569934666380955735263664889604798016194035704361047493027641699022507373990773216443687431071760958198437503246519811635672063448591496
```

`generate.py`
```python
from cryptography.hazmat.primitives.kdf.hkdf import HKDF
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.backends import default_backend
from Crypto.Util.number import long_to_bytes, bytes_to_long, getPrime
from random import randint

def hkdf_mask(secret: bytes, length: int) -> bytes:
  hkdf = HKDF(
    algorithm=hashes.SHA256(),
    length=length,
    salt=None,
    info=b"rsa-d-mask",
    backend=default_backend()
  )
  return hkdf.derive(secret)

# RSA
q1 = getPrime(512)
q2 = getPrime(512)
n = q1 * q2
e = 65537
phi = (q1 - 1) * (q2 - 1)
d = pow(e, -1, phi)

# DLP
p = 200167626629249973590210748210664315551571227173732968065685194568612605520816305417784745648399324178485097581867501503778073506528170960879344249321872139638179291829086442429009723480288604047975360660822750743411854623254328369265079475034447044479229192540942687284442586906047953374527204596869578972378578818243592790149118451253249
g = 11
s = randint(1, 1 << 100)
A = pow(g, s, p)

D = d ^ bytes_to_long(hkdf_mask(long_to_bytes(s), d.bit_length() // 8))

with open('flag.txt', 'r') as outfile:
  flag = outfile.read()

c = pow(bytes_to_long(flag.encode()), 2, n)

print("---- RSA Public Data ----")
print(f"{n=}")
print(f"{e=}")
print("---- DLP Public Data ----")
print(f"{p=}")
print(f"{g=}")
print(f"{A=}")
print("---- Encrypted Data ----")
print(f"{D=}")
print(f"{c=}")
```

Solve script:
```python
import math, random
from Crypto.Util.number import long_to_bytes, bytes_to_long
from cryptography.hazmat.primitives.kdf.hkdf import HKDF
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.backends import default_backend

n = 71016310005824589926747341243598522145452505235842335510488353587223142066921470760443852767377534776713566052988373656012584808377496091765373981120165220471527586994259252074709653090148780742972203779666231432769553199154214563039426087870098774883375566546770723222752131892953195949848583409407713489831
e = 65537
D = 9478993126102369804166465392238441359765254122557022102787395039760473484373917895152043164556897759129379257347258713397227019255397523784552330568551257950882564054224108445256766524125007082113207841784651721510041313068567959041923601780557243220011462176445589034556139643023098611601440872439110251624
c = 1479919887254219636530919475050983663848182436330538045427636138917562865693442211774911655964940989306960131568709021476461747472930022641984797332621318327273825157712858569934666380955735263664889604798016194035704361047493027641699022507373990773216443687431071760958198437503246519811635672063448591496

s = 485391067385099231898174017598

# Step 3: Recover d
def hkdf_mask(secret, length):
    return HKDF(algorithm=hashes.SHA256(), length=length, salt=None,
                info=b'rsa-d-mask', backend=default_backend()).derive(secret)

s_bytes = long_to_bytes(s)
d = None
for mask_len in [128, 127]:
    mask = bytes_to_long(hkdf_mask(s_bytes, mask_len))
    d_cand = D ^ mask
    if pow(pow(2, e, n), d_cand, n) == 2:
        d = d_cand
        print(f'd recovered (mask_len={mask_len}), d bits={d.bit_length()}')
        break

assert d is not None

# Step 4: Factor n using (e,d)
def factor_rsa(n, e, d):
    k = e * d - 1
    while True:
        g_r = random.randint(2, n - 2)
        t = k
        while t % 2 == 0:
            t //= 2
            x = pow(g_r, t, n)
            if x > 1:
                f = math.gcd(x - 1, n)
                if 1 < f < n:
                    return f, n // f

q1, q2 = factor_rsa(n, e, d)
assert q1 * q2 == n
print(f'n factored: q1 bits={q1.bit_length()}, q2 bits={q2.bit_length()}')

# Step 5: Rabin decrypt c = m^2 mod n
def sqrt_mod_p(a, p):
    if p % 4 == 3:
        return pow(a, (p+1)//4, p)
    q, s = p-1, 0
    while q % 2 == 0: q //= 2; s += 1
    z = 2
    while pow(z,(p-1)//2,p) != p-1: z += 1
    M, c_, t, R = s, pow(z,q,p), pow(a,q,p), pow(a,(q+1)//2,p)
    while True:
        if t == 1: return R
        i, tmp = 1, pow(t,2,p)
        while tmp != 1: tmp = pow(tmp,2,p); i += 1
        b = pow(c_, 1 << (M-i-1), p)
        M, c_, t, R = i, pow(b,2,p), t*pow(b,2,p)%p, R*b%p

rp = sqrt_mod_p(c % q1, q1)
rq = sqrt_mod_p(c % q2, q2)
inv_q = pow(q2, -1, q1)
inv_p = pow(q1, -1, q2)

print('Checking 4 Rabin square roots:')
for sp, sq in [(rp,rq),(rp,q2-rq),(q1-rp,rq),(q1-rp,q2-rq)]:
    root = (sp * q2 * inv_q + sq * q1 * inv_p) % n
    try:
        b = long_to_bytes(root)
        t = b.decode('latin-1')
        print(f'  {t!r}')
    except: pass
```

Output:
```shell
python3 solve.py  
d recovered (mask_len=127), d bits=1020  
n factored: q1 bits=512, q2 bits=512  
Checking 4 Rabin square roots:  
"\x12'zye\x98\x07µ\x0e\x9diÓÅ\x9f1\n\x9f*\x96Ï\x99+QxLÑe/òÄ\x94\x83xµ\x9fdOJ\x00\x06â+gèÍ&ü~+f¬\x0f\x0b9\x9a¥HÍüqÀ²\x84ÆÁ:\t£\x95ä\x84PoË[6fKöÝ²ª\x83Vâ<\x01÷Ì\rÐf\x9aú}\x1b¼a+EþeîÁÄ\x1f{ÄB\tÓGP\x99.Ì\x9d¿ðQà¿\x82°ò\x19\x92ë"  
'e!mØIðf|(C7z»»=\tk\x01FÊ\x0böøT\x00Ê0{g¸kd\x94ú\tÆ$Âwðî\xad(µ\x11\x0b@{\x14j\x12\x98\x8b\x06¹\x1c_ãâ0Þë£J1ëÒì©\tOo§3ùèi\'\x83vÞë\x1e«,\xa0}§Kæ\x83F\x88tÇ\x9fú#\x10²\x12p+"\x82víåµLÐÛu\x87;®\x18v\x10¿\x14¬\x1c\x19\x92q¶\x9d'  
'gigem{100lsb_ed_fact0ring_rab1n_attack_1n_th3_log_3oVAjvoCTGmWg847g9zsNBIyPPWqYdP}\n'  
'Rùó^äX^Ç\x19¥Í¦ö\x1c\x0bþËÖ¯úrË¦Û³øËKtóÖá\x1cDjaÕxwê\x0c\x81ÀÌDK\xaddNpáº¯ý\x8aêyuK#}\x9f\x7fæäâ;²\x81\x8c*\x91\x98ÊÐ b=\x01\r\x8d¤\x06³{ÒÛ#è\x0c\x12L\\á©·\xad\x18&Ö\x8ay\x7fµ\x05ÄÉ\x88«w4û^h\x80/¼ÿ\x99½\x84D\nÂ\x04¨\xa0¼'
```

`gigem{100lsb_ed_fact0ring_rab1n_attack_1n_th3_log_3oVAjvoCTGmWg847g9zsNBIyPPWqYdP}`

___

## Abnormal Ellipse

**Category**: crypto

**Author**: `Marfung37`

**Description**: Some reason people are using these weird ellipses to do their encryption. But aren't quadratic curves completely broken. Anyways, I caught the data that was being sent along with how they are doing the encryption.

I get the challenge files:
```shell
cat data.txt  
G=(46876917648549268272641716936114495226812126512396931121066067980475334056759 : 29018161638760518123770904309639572979634020954930188106398864033161780615057 : 1)  
PA=(41794565872898552028378254333448511042514164360566217446125286680794907163222 : 28501067479064047326107608780246105661757692260405498327414296914217192089882 : 1)  
PB=(832923623940209904267388169663314834051489004894067103155141367420578675552 : 7382962163953851721569729505742450736497607615866914193411926051803583826592 : 1)  
Encrypted data: e31e0e638110d1e5c39764af90ac6194c1f9eaabd396703371dc2e6bb2932a18d824d86175ab071943cba7c093ccc6c6  
IV: 478876e42be078dceb3aee3a6a8f260f
```

```shell
cat generate.sage  
from sage.all import *  
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes  
from cryptography.hazmat.backends import default_backend  
from cryptography.hazmat.primitives import padding  
from random import randint  
import hashlib  
import os  
  
p = 57896044618658103051097247842201434560310253892815534401457040244646854264811  
  
# y^2 = x^3 + 57896044618658103051097247842201434560310253892815534336455328262589759096811*x + 6378745995050415640528904257536000  
a = 57896044618658103051097247842201434560310253892815534336455328262589759096811  
b = 6378745995050415640528904257536000  
E = EllipticCurve(GF(p), [a, b])  
  
# ECDH  
G = E.random_point()  
print(f"{G=}")  
dA = randint(2, G.order())  
dB = randint(2, G.order())  
  
PA = dA * G  
PB = dB * G  
  
print(f"{PA=}")  
print(f"{PB=}")  
  
s = int((dB * PA).x())  
key = hashlib.sha256(int(s).to_bytes((s.bit_length() + 7) // 8, 'big')).digest()  
  
# AES encryption  
with open('flag.txt', 'rb') as infile:  
flag = infile.read()  
  
iv = os.urandom(16)  
cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())  
encryptor = cipher.encryptor()  
  
padder = padding.PKCS7(128).padder()  
padded_data = padder.update(flag) + padder.finalize()  
  
encrypted = encryptor.update(padded_data) + encryptor.finalize()  
  
print('Encrypted data:', encrypted.hex())  
print('IV:', iv.hex())
```

Explaination from AI (helps me with the crypto challenges):
```
The key clue is in the challenge name:"Abnormal Ellipse" = Anomalous Curve - a curve where #E(Fₚ) = p. This makes it vulnerable to Smart's Attack, which reduces the ECDLP to simple division in Z/pZ using the p-adic logarithm.
```

I go to [https://sagecell.sagemath.org/](https://sagecell.sagemath.org/) and run this:
```python
from sage.all import *
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import padding
import hashlib

# Curve parameters
p = 57896044618658103051097247842201434560310253892815534401457040244646854264811
a = 57896044618658103051097247842201434560310253892815534336455328262589759096811
b = 6378745995050415640528904257536000

E  = EllipticCurve(GF(p), [a, b])
Gx = 46876917648549268272641716936114495226812126512396931121066067980475334056759
Gy = 29018161638760518123770904309639572979634020954930188106398864033161780615057
G  = E(Gx, Gy)

PAx = 41794565872898552028378254333448511042514164360566217446125286680794907163222
PAy = 28501067479064047326107608780246105661757692260405498327414296914217192089882
PA  = E(PAx, PAy)

PBx = 832923623940209904267388169663314834051489004894067103155141367420578675552
PBy = 7382962163953851721569729505742450736497607615866914193411926051803583826592
PB  = E(PBx, PBy)

# Sanity check: anomalous?
print(f"#E = {E.order()}")
print(f"p  = {p}")
print(f"Anomalous: {E.order() == p}")

# Smart's Attack
def smart_attack(P, Q, p):
    """
    Solve Q = k*P on an anomalous curve (#E = p) via the p-adic / formal-group lift.
    Returns k such that k*P == Q.
    """
    E  = P.curve()
    Fp = GF(p)

    Eqp = EllipticCurve(Qp(p, 2), [ZZ(c) for c in E.a_invariants()])

    def lift(Pt):
        x0 = ZZ(Pt[0])
        for lift_pt in Eqp.lift_x(x0, all=True):
            if ZZ(lift_pt[1]) % p == ZZ(Pt[1]):
                return lift_pt
        raise ValueError("No valid lift found")

    P_lift = lift(P)
    Q_lift = lift(Q)

    pP = p * P_lift
    pQ = p * Q_lift

    phi_P = -(pP[0] / pP[1])
    phi_Q = -(pQ[0] / pQ[1])

    k = ZZ(phi_Q / phi_P) % p
    assert k * P == Q, "Smart's attack failed - curve may not be anomalous :("
    return k

# Recover private key and shared secret
print("\nRunning Smart's attack to recover dA ...")
dA = smart_attack(G, PA, p)
print(f"dA = {dA}")

S  = dA * PB          # shared secret point
s  = int(S[0])        # x-coordinate
print(f"Shared secret x = {s}")

# Derive AES key
key = hashlib.sha256(s.to_bytes((s.bit_length() + 7) // 8, 'big')).digest()

# Decrypt
ciphertext = bytes.fromhex(
    "e31e0e638110d1e5c39764af90ac6194c1f9eaabd396703371dc2e6bb2932a18"
    "d824d86175ab071943cba7c093ccc6c6"
)
iv = bytes.fromhex("478876e42be078dceb3aee3a6a8f260f")

cipher    = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
decryptor = cipher.decryptor()
padded    = decryptor.update(ciphertext) + decryptor.finalize()

unpadder  = padding.PKCS7(128).unpadder()
flag      = unpadder.update(padded) + unpadder.finalize()
print(f"\nFlag: {flag.decode()}")
```

Output:
```shell
#E = 57896044618658103051097247842201434560310253892815534401457040244646854264811
p  = 57896044618658103051097247842201434560310253892815534401457040244646854264811
Anomalous: True

Running Smart's attack to recover dA ...
dA = 5302515257459728333067206555460709176819601641952986275899160992587299740102
Shared secret x = 46862424626771023060842312162194505004189661568357160582950231202292115713415

Flag: gigem{an0ma1ou5_curv3_ss5a_d41z8GaFF3kZ8}
```

`gigem{an0ma1ou5_curv3_ss5a_d41z8GaFF3kZ8}`

___

## Random Password

**Category**: crypto

**Author**: `Marfung37`

**Description**: Password verification by sleepy counting

I get the challenge files. `solver-template.py`:
```python  
from pwn import *  
  
context.log_level = "debug"  
io = remote("streams.tamuctf.com", 443, ssl=True, sni="random-password")  
io.interactive(prompt="")
```

`server.py`:
```python
from functools import partial
import re
import random

random.seed(121728)

def random_sleep(timeout: float) -> None:
  time_elapsed = 0
  while time_elapsed < timeout:
    time_elapsed += random.random()

handle_zero = partial(random_sleep, 5)
handle_one = partial(random_sleep, 17)

def verify(password: str, correct_value: float) -> bool:
  if len(password) != 64: return False

  bit_str = bin(int(password, 16))[2:].zfill(256)

  for bit in bit_str:
    match bit:
      case '0':
        handle_zero()
      case '1':
        handle_one()

  return random.random() == correct_value

password = input('Enter the password in hex: ')
if not re.match('^[0-9a-f]+$', password):
  print('Invalid nonhex input')
  exit()

if verify(password, 0.9992610559813815):
  with open('flag.txt', 'r') as outfile:
    print("Here's the flag", outfile.readline().strip())
    exit()
else:
  print('Incorrect password')
  exit()
```

The server starts with: `random.seed(121728)`. Pythons RNG will now produce the exact same sequence every run. 

The password must be 64 hex chars (256 bits): 
```python
bit_str = bin(int(password, 16))[2:].zfill(256)
```

After all 256 bits are processed:
```python
return random.random() == 0.9992610559813815
```
This means the next RNG value must equal this constant. So we want the RNG pointer to land exactly on the index where this value occurs.

Solve script:
```python
from pwn import *
import random
from collections import deque

TARGET = 0.9992610559813815
SEED = 121728
BITS = 256

random.seed(SEED)

# generate lots of random numbers
nums = [random.random() for _ in range(20000)]

# find target index
target_index = nums.index(TARGET)
print("Target RNG index:", target_index)

def draws_needed(start, timeout):
    s = 0
    i = start
    while s < timeout:
        s += nums[i]
        i += 1
    return i - start

# precompute transitions
k0 = {}
k1 = {}

for i in range(target_index + 1):
    k0[i] = draws_needed(i, 5)
    k1[i] = draws_needed(i, 17)

# BFS search
queue = deque([(0, 0, "")])  # step, index, bits
seen = set()

while queue:
    step, idx, bits = queue.popleft()

    if step == BITS:
        if idx == target_index:
            print("FOUND!")
            print(bits)
            break
        continue

    state = (step, idx)
    if state in seen:
        continue
    seen.add(state)

    # bit 0
    nidx = idx + k0[idx]
    if nidx <= target_index:
        queue.append((step+1, nidx, bits+"0"))

    # bit 1
    nidx = idx + k1[idx]
    if nidx <= target_index:
        queue.append((step+1, nidx, bits+"1"))

bitstring = bits
password = hex(int(bitstring, 2))[2:].zfill(64)

print("Password:", password)

context.log_level = "debug"
io = remote("streams.tamuctf.com", 443, ssl=True, sni="random-password")
io.interactive(prompt="")
```

Output:
```shell
python3 solve.py
Target RNG index: 5719  
FOUND!  
0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000011111111111111111111111111111111111011111111111111111111111111111111111111111111111111111111111111111111111111111111111101111111  
Password: 00000000000000000000000000000000ffffffffefffffffffffffffffffff7f  
[+] Opening connection to streams.tamuctf.com on port 443: Done  
[*] Switching to interactive mode  
[DEBUG] Received 0x1b bytes:  
b'Enter the password in hex: '  
Enter the password in hex: 00000000000000000000000000000000ffffffffefffffffffffffffffffff7f  
[DEBUG] Sent 0x41 bytes:  
b'00000000000000000000000000000000ffffffffefffffffffffffffffffff7f\n'  
[DEBUG] Received 0x36 bytes:  
b"Here's the flag gigem{h3rd1ng_rand0m_sh3ep_LiNBpqRTk}\n"  
Here's the flag gigem{h3rd1ng_rand0m_sh3ep_LiNBpqRTk}
```

`gigem{h3rd1ng_rand0m_sh3ep_LiNBpqRTk}`
