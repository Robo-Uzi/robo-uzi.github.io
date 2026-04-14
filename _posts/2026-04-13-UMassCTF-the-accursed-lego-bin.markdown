---
layout: post
title:  "The Accursed Lego Bin"
date:   2026-04-13 18:52:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UMassCTF-2026-the-accursed-lego-bin/
---

**Category**: crypto easy

**Author**: Sam

**Description**: I dropped my message into the bin of Legos. It's all scrambled up now. Please help.

I get the challenge files. `encoder.py`:
```python
import random
import time
import dotenv
from Cryptodome.Util.number import getPrime
from Cryptodome.PublicKey import RSA

try:
    dotenv.load_dotenv()
    flag = dotenv.get_key(".env", "FLAG")
except KeyError:
    flag = "UMASS{simple_test_flag}"

e = 7

def RSA_enc(plain_text):
    p, q = getPrime(2048), getPrime(2048)
    n = p * q
    plain_num = int.from_bytes(plain_text.encode(), "big")
    ciphertext = pow(plain_num, e, n)
    return n, ciphertext

def get_flag_bits(flag):
    flag_bits = []
    for char in flag:
        char_bits = bin(ord(char))[2:].zfill(8)
        flag_bits.extend(char_bits)
    return flag_bits

def bit_arr_to_str(bit_arr):
    byte_arr = []
    for i in range(0, len(bit_arr), 8):
        byte = bit_arr[i:i+8]
        char = int(''.join(byte), 2)
        byte_arr.append(char)
    bytes_arr = bytes(byte_arr)
    return bytes.hex(bytes_arr)


def main():
    text = "I_LOVE_RNG"
    n, seed = RSA_enc(text)
    flag_bits = get_flag_bits(flag)
    for i in range(10):
        random.seed(seed*(i+1))
        random.shuffle(flag_bits)
    # now output the shuffled flag as binary
    enc_flag_bytes = bit_arr_to_str(flag_bits)
    enc_seed = pow(seed, e, n)
    with open("output.txt", "w") as f:
        f.write(f"seed = {enc_seed}\n")
        f.write(f"flag = {enc_flag_bytes}\n")

if __name__ == "__main__":
    main()
```

`output.txt`:
```shell
seed = 27853968878118202600616227164274184566757028924504378904793832254042520819991144639702067205911203237440164930417495337197532501173607130020895075421529488925453640401673956438276491981209692168887241600331323119747563338336714474549971016558306628074198388772585672217715120627041791075104601103026751194857235765309608359123653353317678322176850235969280946203083455072140605141795053378439195293814791874092411691470992665912679118059266672118104677436338717139016415491690881114160151442145485980845723522027034166250144387200630948484934412980402141190370298072772878692178174395473352346736568834853932546775351591579301264010616662074516876263415244325179769805404580595987957830206775099221681479552297343673953519347816803686755315058241114932909715588571465125584675910868587612361307253375806962785674201551995414052898626175776112925401104907258409223265509906782478388392655489350014728299523474441953620142576405825798349964376116586305354010422094308152856531053593521850744465605649669069637606613192817098670399196448110611736116364403445860585755736974514672765253945103150765043635481842335038685418842068710568699703147745504514090439  
flag = a9fa3c5e51d4cea498554399848ad14aa0764e15a6a2110b6613f5dc87fa70f17fafbba7eb5a2a5179
```

- `"I_LOVE_RNG"` is only 80 bits long
- With `e=7` plaintext^7 is at most 560 bits
- But `n` is 4096 bits so `pow(plaintext, 7, n) == pow(plaintext, 7)`. No modular reduction happens.
- This means `enc_seed = seed^7` as a plain integer, so we recover `seed` via an integer 7th root

Pythons `random.shuffle` uses Fisher-Yates which makes a deterministic sequence of swaps given a seed. Replay those exact swaps in reverse order from 9 down to 0.

Solve script:
```python
from gmpy2 import iroot
import random

enc_seed = 27853968878118202600616227164274184566757028924504378904793832254042520819991144639702067205911203237440164930417495337197532501173607130020895075421529488925453640401673956438276491981209692168887241600331323119747563338336714474549971016558306628074198388772585672217715120627041791075104601103026751194857235765309608359123653353317678322176850235969280946203083455072140605141795053378439195293814791874092411691470992665912679118059266672118104677436338717139016415491690881114160151442145485980845723522027034166250144387200630948484934412980402141190370298072772878692178174395473352346736568834853932546775351591579301264010616662074516876263415244325179769805404580595987957830206775099221681479552297343673953519347816803686755315058241114932909715588571465125584675910868587612361307253375806962785674201551995414052898626175776112925401104907258409223265509906782478388392655489350014728299523474441953620142576405825798349964376116586305354010422094308152856531053593521850744465605649669069637606613192817098670399196448110611736116364403445860585755736974514672765253945103150765043635481842335038685418842068710568699703147745504514090439

enc_flag_hex = "a9fa3c5e51d4cea498554399848ad14aa0764e15a6a2110b6613f5dc87fa70f17fafbba7eb5a2a5179"

# integer 7th root
seed, exact = iroot(enc_seed, 7)
print(f"Exact: {exact}, plaintext: {int(seed).to_bytes((int(seed).bit_length()+7)//8, 'big')}")

# flag bits
enc_flag_bytes = bytes.fromhex(enc_flag_hex)
flag_bits = []
for byte in enc_flag_bytes:
    flag_bits.extend(list(bin(byte)[2:].zfill(8)))

# reverse shuffles
def reverse_shuffle(arr, seed_val):
    n = len(arr)
    rng = random.Random(seed_val)
    swaps = []
    for i in reversed(range(1, n)):
        j = rng.randrange(i + 1)
        swaps.append((i, j))
    for i, j in reversed(swaps):
        arr[i], arr[j] = arr[j], arr[i]
    return arr

for i in reversed(range(10)):
    flag_bits = reverse_shuffle(flag_bits, int(seed) * (i + 1))

# bits to string
byte_arr = []
for i in range(0, len(flag_bits), 8):
    char = int(''.join(flag_bits[i:i+8]), 2)
    byte_arr.append(char)

flag = bytes(byte_arr).decode('utf-8', errors='replace')
print(f"Flag: {flag}")
```

Output:
```shell
python3 solve3.py  
Exact: True, plaintext: b'\nig\xe1\xb1T\xb8q\x1bW#;\x98M\ny\x9cgH\xba6\x19\xed\x07\x1br\xf7T\xc2\x9bU.\xceh\x12\xc4==5\xec\x85\xcc\xc8rh\xd5s\x9d)2\x0bd\xbd\x98\xc7Q\xbe\x84\xe3\x93\xc0\x927\xba\xb4\x02\xe2@\xb7'  
Flag: UMASS{tH4Nk5_f0R_uN5CR4m8L1nG_mY_M3554g3}
```

`UMASS{tH4Nk5_f0R_uN5CR4m8L1nG_mY_M3554g3}`
