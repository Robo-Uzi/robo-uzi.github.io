---
layout: post
title:  "Cascader"
date:   2025-09-15 13:10:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /fortid-2025-cascader/
---

**Title:** Cascader

**Category:** crypto

**Description:** Just found this super cool key exchange protocol while scrolling Hacker News between meetings ☕️⚡️🧠 It's based on some clean recurrence math — none of that dinosaur-era number theory stuff 🦕 finally, crypto that doesn't look like it was invented in the 70s 📟 It came with a working implementation too, so i plugged it right in and shipped to prod 🚀🛠️ A few people said I should’ve used something more “proven” 🔒🤷‍♂️ but honestly... this just feels right ✨ Anyway, i packaged it into a challenge. Curious to see what the skeptics say now 😏🔍

I am given 3 files, `output.txt`, `chall.js`, and `cascader.pdf`.

Abstract given in the pdf:
```
Abstract. Cascader, a novel key-exchange protocol based on an iter-
ative multiplicative recurrence over a finite field, is introduced. In con-
trast to standard methods, e.g., traditional Diffie–Hellman and ECC, it
replaces exponentiation and scalar multiplication with layered products,
achieving commutativity and deterministic pseudorandom behavior.
```
The pdf explains the math behind the protocol. I search for the title and find the link to the original here: [Cascader: A Recurrence-Based Key Exchange Protocol](https://eprint.iacr.org/2025/1304.pdf). 

With the same search I see a paper for shared key recovery: [Shared Key Recovery Attack on Cascader Key Exchange Protocol](https://eprint.iacr.org/2025/1418.pdf):
```shell
Shared Key Recovery Attack
In a secure key exchange protocol it should be hard for an attacker 
to recover the shared secret, this implies that it should also be 
hard to recover the private key from the public key. Although we have 
not found a way to recover the private key from the public key, we do 
show that the shared secret between Alice and Bob can be recovered using 
only public values. Since Alice calculates the shared secret as S = B * P(a), 
knowledge of a is not necessary when P(a) is known. P(a) can be recovered 
from the public key A by using the modular multiplicative inverse of 
SEED, which can be calculated using the extended Euclidean algorithm. 
The shared secret can thus be calculated as follows:

P(a) = A * SEED^−1 mod p
S = B * P(a) mod p

Appending the code in listing 1 after the reference implementation from [Lin25] will print the 
shared secret between Alice and Bob.

function modInv(a, b) {
return 1n < a ? b - modInv(b % a, a) * b / a : 1n;
}
console.log(bobPublic * modInv(SEED, MOD) * alicePublic % MOD)

Listing 1: Shared key recovery attack against the reference implementation using the 
multiplicative modular inverse of SEED [Dav16].
```

`chall.js`:
```shell
"use strict";  
  
const { createHash, createCipheriv, randomBytes } = require('node:crypto');  
  
const KEY_SIZE_BITS = 256n;  
const MAX_INT = 1n << KEY_SIZE_BITS;  
const MOD = MAX_INT - 189n; // Prime number  
const SEED = MAX_INT / 5n;  
  
function linearRecurrence(seed, exponents) {  
   let result = seed;  
   let exp = 1n;  
   while (exponents > 0n) {  
       if (exponents % 2n === 1n) {  
           let mult = 1n;  
           for (let i = 0; i < exp; i++) {  
               result = 3n * result * mult % MOD;  
               mult <<= 1n;  
           }  
       }  
       exponents >>= 1n;  
       exp++;  
   }  
   return result;  
}  
// Generate a random 256 - bit BigInt  
function random256BitBigInt() {  
   const array = new Uint8Array(32);  
   crypto.getRandomValues(array);  
   let hex = '0x';  
   for (const byte of array) {  
       hex += byte.toString(16).padStart(2, '0 ');  
   }  
   return BigInt(hex);  
}  
const alicePrivate = random256BitBigInt();  
const bobPrivate = random256BitBigInt();  
const alicePublic = linearRecurrence(SEED, alicePrivate);  
const bobPublic = linearRecurrence(SEED, bobPrivate);  
const aliceShared = linearRecurrence(bobPublic,  
   alicePrivate);  
const bobShared = linearRecurrence(alicePublic, bobPrivate);  
console.log("Alice private ", alicePrivate.toString());  
console.log("Bob private ", bobPrivate.toString());  
console.log("Alice public ", alicePublic.toString());  
console.log("Bob public ", bobPublic.toString());  
console.log("Alice Shared ", aliceShared.toString());  
console.log("Bob Shared ", bobShared.toString());  
console.log("Alice's and Bob's shared secrets equal? ",  
   aliceShared === bobShared);  
  
function bigIntToFixedBE(n, lenBytes) {  
 let hex = n.toString(16);  
 if (hex.length % 2) hex = "0" + hex;  
 const buf = Buffer.from(hex, "hex");  
 if (buf.length > lenBytes) {  
   return buf.slice(-lenBytes);  
 } else if (buf.length < lenBytes) {  
   const pad = Buffer.alloc(lenBytes - buf.length, 0);  
   return Buffer.concat([pad, buf]);  
 }  
 return buf;  
}  
  
function sha256(buf) {  
 return createHash("sha256").update(buf).digest();  
}  
  
function encryptAESGCM(key, plaintext) {  
  const iv = randomBytes(12);  
  const cipher = createCipheriv('aes-256-gcm', key, iv);  
  const ciphertext = Buffer.concat([cipher.update(plaintext, 'utf8'), cipher.final()]);  
  const tag = cipher.getAuthTag();  
  return { iv, ciphertext, tag };  
}  
  
const sharedBytes = bigIntToFixedBE(aliceShared, 32);  
const aesKey = sha256(sharedBytes);  
  
const FLAG = "FortID{<REDACTED>}"  
  
const { iv, ciphertext, tag } = encryptAESGCM(aesKey, FLAG);  
console.log("ct (hex):   ", Buffer.concat([iv, ciphertext, tag]).toString("hex"));
```
It sets up a 256 bit modulus (`MOD = 2^256 - 189`), and a fixed public seed (`SEED = 2^256 / 5`). `linearRecurrence(seed, exponents)` repeatedly multiplies the seed based on the bits of `exponents`. The result is always the seed times some function of `exponents`.

It generates two 256 bit private numbers (Alice and Bob), computes their public values by running `linearRecurrence(SEED, private)`, and each side computes the shared secret by running `linearRecurrence(other_public, own_private)`. It derives an AES key by hashing the shared secret and encrypts FLAG with AES-GCM. 

Knowing SEED and both public values means you can recover the shared secret.

`output.txt`:
```shell
cat output.txt  
Alice private  <REDACTED>  
Bob private  <REDACTED>  
Alice public 81967497404473670873986762408662347640688858544889917659709378751872081150739  
Bob public 25638634989672271296647305730621408042240305773269414164982933528002524403752  
Alice Shared  <REDACTED>  
Bob Shared  <REDACTED>  
Alices and Bobs shared secrets equal?  true  
ct (hex): e2f84b71e84c8d696923702ddb1e35993e9108289e2d14ae8f05441ad48d1a67ead74f5f230d39dbfaae5709448c2690237ac6ab88fc26c8f362284d1e8063491d63f7c15cc3b024c62b5069605b73dd2c54fdcb2823c0c235b20e52dc5630c5f3
```

Solve script:
```python
from hashlib import sha256
from Crypto.Cipher import AES

MOD = 2**256 - 189
SEED = 2**256 // 5

alice_public = 81967497404473670873986762408662347640688858544889917659709378751872081150739
bob_public   = 25638634989672271296647305730621408042240305773269414164982933528002524403752

ct_hex = "e2f84b71e84c8d696923702ddb1e35993e9108289e2d14ae8f05441ad48d1a67ead74f5f230d39dbfaae5709448c2690237ac6ab88fc26c8f362284d1e8063491d63f7c15cc3b024c62b5069605b73dd2c54fdcb2823c0c235b20e52dc5630c5f3"
ct_bytes = bytes.fromhex(ct_hex)
iv = ct_bytes[:12]
ciphertext = ct_bytes[12:-16]
tag = ct_bytes[-16:]

# find shared secret directly
shared_secret = (alice_public * bob_public * pow(SEED, -1, MOD)) % MOD

# convert to 32 bytes
shared_bytes = shared_secret.to_bytes(32, 'big')
aes_key = sha256(shared_bytes).digest()

# Decrypt AES-GCM
cipher = AES.new(aes_key, AES.MODE_GCM, nonce=iv)
cipher.update(b"")
flag = cipher.decrypt_and_verify(ciphertext, tag)
print("Flag:", flag.decode())
```

`FortID{St0p_B31n6_4_H1ps73r_4nd_5t1ck_70_Th3_G00d_0ld_D1ff1e_H3l1man}`