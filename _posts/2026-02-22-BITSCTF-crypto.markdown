---
layout: post
title:  "Crypto Challenges"
date:   2026-02-22 18:11:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /BITSCTF-2026-crypto-challenges/
---
* TOC
{:toc}

## Aliens Eat Snacks

**Author**: Aur0r4

**Category**: crypto

**Description**: I found a very secure algorith online and used it to encrypt the flag. I heard it's very secure so I used it to encrypt the flag.

I get the challenge files. Contents of `README.md:
```markdown
# AES  
  
I found an AES implementation online and used it to encrypt the flag. I heard AES is very secure so I used it to encrypt the flag.  
  
## Files  
  
- `aes.py` - The AES implementation  
- `output.txt` - Some encrypted data for you to analyze  
  
## Flag Format  
  
`BITSCTF{...}`
```

Contents of `aes.py`:
```python
#!/usr/bin/env python3

from typing import List

IRREDUCIBLE_POLY = 0x11B

def gf_mult(a: int, b: int) -> int:
    result = 0
    for _ in range(8):
        if b & 1:
            result ^= a
        hi_bit = a & 0x80
        a = (a << 1) & 0xFF
        if hi_bit:
            a ^= (IRREDUCIBLE_POLY & 0xFF)
        b >>= 1
    return result

def gf_pow(base: int, exp: int) -> int:
    if exp == 0:
        return 1
    result = 1
    while exp > 0:
        if exp & 1:
            result = gf_mult(result, base)
        base = gf_mult(base, base)
        exp >>= 1
    return result

def gf_inv(a: int) -> int:
    if a == 0:
        return 0
    return gf_pow(a, 254)

def generate_sbox() -> List[int]:
    sbox = []
    for x in range(256):
        val = gf_pow(x, 23)
        val ^= 0x63
        sbox.append(val)
    return sbox

def generate_inv_sbox(sbox: List[int]) -> List[int]:
    inv_sbox = [0] * 256
    for i, v in enumerate(sbox):
        inv_sbox[v] = i
    return inv_sbox

SBOX = generate_sbox()
INV_SBOX = generate_inv_sbox(SBOX)

MIX_MATRIX = [
    [0x02, 0x03, 0x01, 0x01],
    [0x01, 0x02, 0x03, 0x01],
    [0x01, 0x01, 0x02, 0x03],
    [0x03, 0x01, 0x01, 0x02]
]

INV_MIX_MATRIX = [
    [0x0E, 0x0B, 0x0D, 0x09],
    [0x09, 0x0E, 0x0B, 0x0D],
    [0x0D, 0x09, 0x0E, 0x0B],
    [0x0B, 0x0D, 0x09, 0x0E]
]

RCON = [0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x80, 0x1B, 0x36]

def key_expansion(key: bytes, rounds: int = 6) -> List[bytes]:
    assert len(key) == 16
    words = []
    for i in range(4):
        words.append(list(key[4*i:4*i+4]))
    
    for i in range(4, 4 * (rounds + 1)):
        temp = words[i-1][:]
        if i % 4 == 0:
            temp = temp[1:] + temp[:1]
            temp = [SBOX[b] for b in temp]
            temp[0] ^= RCON[(i // 4) - 1]
        words.append([words[i-4][j] ^ temp[j] for j in range(4)])
    
    round_keys = []
    for r in range(rounds + 1):
        rk = bytes()
        for i in range(4):
            rk += bytes(words[r*4 + i])
        round_keys.append(rk)
    
    return round_keys

def sub_bytes(state: List[List[int]]) -> List[List[int]]:
    return [[SBOX[state[r][c]] for c in range(4)] for r in range(4)]

def inv_sub_bytes(state: List[List[int]]) -> List[List[int]]:
    return [[INV_SBOX[state[r][c]] for c in range(4)] for r in range(4)]

def shift_rows(state: List[List[int]]) -> List[List[int]]:
    result = [[0]*4 for _ in range(4)]
    for r in range(4):
        for c in range(4):
            result[r][c] = state[r][(c + r) % 4]
    return result

def inv_shift_rows(state: List[List[int]]) -> List[List[int]]:
    result = [[0]*4 for _ in range(4)]
    for r in range(4):
        for c in range(4):
            result[r][c] = state[r][(c - r) % 4]
    return result

def mix_columns(state: List[List[int]]) -> List[List[int]]:
    result = [[0]*4 for _ in range(4)]
    for c in range(4):
        for r in range(4):
            val = 0
            for i in range(4):
                val ^= gf_mult(MIX_MATRIX[r][i], state[i][c])
            result[r][c] = val
    return result

def inv_mix_columns(state: List[List[int]]) -> List[List[int]]:
    result = [[0]*4 for _ in range(4)]
    for c in range(4):
        for r in range(4):
            val = 0
            for i in range(4):
                val ^= gf_mult(INV_MIX_MATRIX[r][i], state[i][c])
            result[r][c] = val
    return result

def add_round_key(state: List[List[int]], round_key: bytes) -> List[List[int]]:
    result = [[0]*4 for _ in range(4)]
    for r in range(4):
        for c in range(4):
            result[r][c] = state[r][c] ^ round_key[r + 4*c]
    return result

def bytes_to_state(data: bytes) -> List[List[int]]:
    state = [[0]*4 for _ in range(4)]
    for i in range(16):
        state[i % 4][i // 4] = data[i]
    return state

def state_to_bytes(state: List[List[int]]) -> bytes:
    result = []
    for c in range(4):
        for r in range(4):
            result.append(state[r][c])
    return bytes(result)

class AES:
    ROUNDS = 4
    
    def __init__(self, key: bytes):
        if len(key) != 16:
            raise ValueError
        self.key = key
        self.round_keys = key_expansion(key, self.ROUNDS)
    
    def encrypt(self, plaintext: bytes) -> bytes:
        if len(plaintext) != 16:
            raise ValueError
        
        state = bytes_to_state(plaintext)
        state = add_round_key(state, self.round_keys[0])
        
        for r in range(1, self.ROUNDS):
            state = sub_bytes(state)
            state = shift_rows(state)
            state = mix_columns(state)
            state = add_round_key(state, self.round_keys[r])
        
        state = sub_bytes(state)
        state = shift_rows(state)
        state = add_round_key(state, self.round_keys[self.ROUNDS])
        
        return state_to_bytes(state)
    
    def decrypt(self, ciphertext: bytes) -> bytes:
        if len(ciphertext) != 16:
            raise ValueError
        
        state = bytes_to_state(ciphertext)
        state = add_round_key(state, self.round_keys[self.ROUNDS])
        state = inv_shift_rows(state)
        state = inv_sub_bytes(state)
        
        for r in range(self.ROUNDS - 1, 0, -1):
            state = add_round_key(state, self.round_keys[r])
            state = inv_mix_columns(state)
            state = inv_shift_rows(state)
            state = inv_sub_bytes(state)
        
        state = add_round_key(state, self.round_keys[0])
        return state_to_bytes(state)
```

Contents of `output.txt`:
```shell
cat output.txt | wc -l  
1004

cat output.txt | head  
key_hint: 26ab77cadcca0ed41b03c8f2e5  
encrypted_flag: 8e70387dc377a09cbc721debe27c468157b027e3e63fe02560506f70b3c72ca19130ae59c6eef47b734bb0147424ec936fc91dc658d15dee0b69a2dc24a78c44  
num_samples: 1000  
samples:  
376f73334dc9db2a4d20734c0783ac69,9070f81f4de789663820e8924924732b  
a4da3590273d7b33b2a4e73210c38a05,f501ed98c671cf1a23e5c028504d2603  
7e52c00ceda7a3c1b338d90721615910,36ddbc3506b0a1844677ceb4006509f1  
c1a10df2e2afc6ee1397ea1422752472,a3ec80650417136efe87d17257edc161  
687bd6c20eb900bc06a2573a9233543f,efa5e5d84d48496206c1d94c98531980  
56f198dc16c838e3afe133f21497fc9b,84d738ce296771708f6bfb307dbc9a30

cat output.txt | tail  
70cbcb99282120789903bf369f9d63a3,6d5d68106988ef392cb66c303aa2114e  
c1a9735b5531b9652e1fdc39fcf27404,12f6b05fa99d3367db92e166d10e3163  
3350ace150237fbda9ce5219ce3d12db,ebaf9f3311e29928b2ccea85257e8023  
abc01f0506912209d4201e193d0d5d59,ad0abf0d6d4e082b3f30680811472339  
016a68e34e21c76ea0440c7fd2f96f23,850dce383164b6d475820f4d1f08b214  
f3f4f120fc1c3aae91a2089022a965a7,9b735a3a1b4de5f0696e32064edd5c4c  
92acff87447f3e5832b56316708af33a,879f59c759cc805db14f848dc4433662  
0839d9b86ec1a9bd7681f4a8a46c67f9,5e88934316cdc84e613573e965c1e07c  
3c0cccad8802109711b9a9404b208f81,b14bce96bf39091bf68aada7e1516ebd  
6133a70a843e7845626414252e7f40eb,67af388f3ee4353a54f0e1a825a051ce
```
I get the first 13 bytes of the 16 byte AES key. Brute forcing is possible because there are only 16,777,216 possibilities for the last 3 bytes. Brute force by testing each candidate key against a few plaintext ciphertext pairs.

Solve script:
```python
#!/usr/bin/env python3

from aes import AES

key_hint = "26ab77cadcca0ed41b03c8f2e5"
key_part = bytes.fromhex(key_hint)

samples = [
    ("376f73334dc9db2a4d20734c0783ac69", "9070f81f4de789663820e8924924732b"),
    ("a4da3590273d7b33b2a4e73210c38a05", "f501ed98c671cf1a23e5c028504d2603"),
    ("7e52c00ceda7a3c1b338d90721615910", "36ddbc3506b0a1844677ceb4006509f1"),
]

pt_list = [bytes.fromhex(pt) for pt, ct in samples]
ct_list = [bytes.fromhex(ct) for pt, ct in samples]

encrypted_flag_hex = (
    "8e70387dc377a09cbc721debe27c468157b027e3e63fe02560506f70b3c72ca1"
    "9130ae59c6eef47b734bb0147424ec936fc91dc658d15dee0b69a2dc24a78c44"
)
encrypted_flag = bytes.fromhex(encrypted_flag_hex)

def find_key(is_prefix=True):
    for i in range(256):
        print(f"Progress for {'prefix' if is_prefix else 'suffix'}: {i}/255")
        for j in range(256):
            for k in range(256):
                suffix_bytes = bytes([i, j, k])
                if is_prefix:
                    key = key_part + suffix_bytes
                else:
                    key = suffix_bytes + key_part
                aes = AES(key)
                match = True
                for idx in range(len(samples)):
                    encrypted = aes.encrypt(pt_list[idx])
                    if encrypted != ct_list[idx]:
                        match = False
                        break
                if match:
                    return key
    return None

# Try prefix first
key = find_key(is_prefix=True)
if key is None:
    print("Not found as prefix, trying as suffix.")
    key = find_key(is_prefix=False)

if key is None:
    print("Key not found.")
else:
    print("Found key:", key.hex())
    aes = AES(key)
    plaintext = b""
    for i in range(0, len(encrypted_flag), 16):
        block = encrypted_flag[i:i+16]
        dec_block = aes.decrypt(block)
        plaintext += dec_block
    print("Decrypted flag:", plaintext.decode('utf-8'))
```

Output of solve script:
```shell
python3 solve.py  
Progress for prefix: 0/255  
Progress for prefix: 1/255  
Progress for prefix: 2/255  
Progress for prefix: 3/255  
Progress for prefix: 4/255  
Progress for prefix: 5/255

... etc etc ...

Progress for prefix: 201/255  
Progress for prefix: 202/255  
Progress for prefix: 203/255  
Progress for prefix: 204/255  
Progress for prefix: 205/255  
Found key: 26ab77cadcca0ed41b03c8f2e5cdec0c  
Decrypted flag: BITSCTF{7h3_qu1ck_br0wn_f0x_jump5_0v3r_7h3_l4zy_d0g}
```
This did take forever to run but it worked. Im sure there is a much better approach.

`BITSCTF{7h3_qu1ck_br0wn_f0x_jump5_0v3r_7h3_l4zy_d0g}`
