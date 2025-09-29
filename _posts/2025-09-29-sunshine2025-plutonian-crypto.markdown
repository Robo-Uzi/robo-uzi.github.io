---
layout: post
title:  "Plutonian Crypto"
date:   2025-09-29 13:10:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /sunshine-2025-plutonian-crypto/
---

**Title:** Plutonian Crypto

**Category:** crypto

**Author:** tsuto

**Description:** One of our deep space listening stations has been receiving a repeating message that appears to be coming from Pluto. It is encrypted with some sort of cipher but our best scientists have at least been able to decrypt the first part of the message, _"Greetings, Earthlings."_

See if you are able to somehow break their encryption and find out what the message is!

`nc chal.sunshinectf.games 25403`

I get `main.py`:
```shell
#!/usr/bin/env python3  
import sys  
import time  
from binascii import hexlify  
from Crypto.Cipher import AES  
from Crypto.Util import Counter  
from Crypto.Random import get_random_bytes  
from ctf_secrets import MESSAGE  
  
KEY = get_random_bytes(16)  
NONCE = get_random_bytes(8)  
  
WELCOME = '''  
    ,-.  
   /   \\    
  :     \\      ....*  
  | . .- \\-----00''  
  : . ..' \\''//  
   \\ .  .  \\/  
    \\ . ' . NASA Deep Space Listening Posts  
 , . \\       \\     ~ Est. 1969 ~  
,|,. -.\\       \\  
   '.|| `-...__..-  
     | | "We're always listening to you!"  
    |__|  
   /||\\\\  
   //||\\\\  
  // || \\\\  
__//__||__\\\\__  
'--------------'  
'''  
   
def main():  
   
   # Print ASCII art and intro  
   sys.stdout.write(WELCOME)  
   sys.stdout.flush()  
   time.sleep(0.5)  
      
   sys.stdout.write("\nConnecting to remote station")  
   sys.stdout.flush()  
      
   for i in range(5):  
       sys.stdout.write(".")  
       sys.stdout.flush()  
       time.sleep(0.5)  
      
   sys.stdout.write("\n\n== BEGINNING TRANSMISSION ==\n\n")  
   sys.stdout.flush()  
  
   C = 0  
   while True:  
       ctr = Counter.new(64, prefix=NONCE, initial_value=C, little_endian=False)  
       cipher = AES.new(KEY, AES.MODE_CTR, counter=ctr)  
       ct = cipher.encrypt(MESSAGE)  
       sys.stdout.write("%s\n" % hexlify(ct).decode())  
       sys.stdout.flush()  
       C += 1  
       time.sleep(0.5)  
  
if __name__ == "__main__":  
   main()
```

When I connect I see this:
```shell
nc chal.sunshinectf.games 25403  
  
    ,-.  
   /   \    
  :     \      ....*  
  | . .- \-----00''  
  : . ..' \''//  
   \ .  .  \/  
    \ . ' . NASA Deep Space Listening Posts  
 , . \       \     ~ Est. 1969 ~  
,|,. -.\       \  
   '.|| `-...__..-  
     | | "We're always listening to you!"  
    |__|  
   /||\\  
   //||\\  
  // || \\  
__//__||__\\__  
'--------------'  
  
Connecting to remote station.....  
  
== BEGINNING TRANSMISSION ==  
  
a4a42216a943d94bf9aaffb5389f43ffb82e5cd235aa374ce288ff5975b560ca12a1ff40d017409986d83785b3d2b3c1cb2a024c0eefbb8d0dbd06d1dfcb469e9f817c48142947e2ba5c0623fc0f4a5543fdd60b74f7a92f73fc94648dcafae622c0cb0832c158c23bf7223e97cbef5dce7f1673e80e5c10898094348407a40314a22e6e8617a765a6df29fcc5f8dc96c240ef6bdf0cbe35f6489a009fc80d507529452761920b77bf5119742051e0ed63a5f7fbdc2fd3c28d2697134557ca2eb563c94ec2cabb26406e6c63468f7653121b5f2b29f52dbc2f8593b183f822975ea30ea71cd6d6b6eb40f8c730bc0f18876289b98746b5023743a7bf51a8da95009d38c99dd10dbf479806413819f01d6f9bb93141eff7d49647a0e930956cc133134172e9d13237ec6c6ee9fab39d40ceb7fa4f8679fc6eea2c762027e26dc50fd0708f2b7a657e4681de43ae4d7fb8bd6a430543c9f8782a360a2cc5cd9fbf4274deb66e2e856eee135640867af99ecaf56a532f5db14bd3092276ef58e67f7ef0b5d50ddb558b26bf3ba2f54ca02b25d0cc463a7758b95fa535ac46af691312...
```

The service is repeatedly transmitting AES-CTR encrypted messages. AES-CTR is a stream cipher mode: every ciphertext line = plaintext XOR keystream. 

The description hints `our best scientists have at least been able to decrypt the first part of the message, "Greetings, Earthlings."`. 

So when my script receives the first ciphertext it will use `"Greetings, Earthlings."` to immediately recover the first 22 bytes of the keystream.

As more messages arrive the script XORs their first 22 bytes with the known prefix, filling in more keystream positions (block, offset). With the keystream reconstructed, I can decrypt any line.

Run this script:
```python
#!/usr/bin/env python3

from pwn import remote
from binascii import unhexlify
import re, sys

HOST, PORT = "chal.sunshinectf.games", 25403
KNOWN      = b"Greetings, Earthlings."
FLAG_RE    = re.compile(rb"sun\{.*?\}")
BLOCK      = 16
TARGET     = 100
PRINT_EVERY= 50

def update_keystream(ks, ct, ctr=0):
    for i, p in enumerate(KNOWN):
        if i >= len(ct): break
        ks[(ctr + i // BLOCK, i % BLOCK)] = ct[i] ^ p

def decrypt(ct, ctr, ks):
    return bytes(ct[i] ^ ks.get((ctr + i // BLOCK, i % BLOCK), ord('?'))
                 for i in range(len(ct)))

def stream(host, port, limit):
    r, cts, ks = remote(host, port, timeout=10), [], {}
    try:
        # skip banner, wait for first hex line
        while True:
            line = r.recvline()
            if not line: continue
            s = line.strip()
            if all(c in b"0123456789abcdefABCDEF" for c in s) and len(s) >= 32:
                ct = unhexlify(s); cts.append(ct)
                update_keystream(ks, ct)
                break

        for idx in range(1, limit):
            line = r.recvline()
            if not line: continue
            s = line.strip()
            if not (all(c in b"0123456789abcdefABCDEF" for c in s) and len(s) >= 32):
                continue
            ct = unhexlify(s); cts.append(ct)
            update_keystream(ks, ct, idx)

            if idx % PRINT_EVERY == 0:
                sys.stdout.write(f"\rCollected {idx}/{limit} lines, ks={len(ks)}"); sys.stdout.flush()

            for j, c in enumerate(cts):
                m = FLAG_RE.search(decrypt(c, j, ks))
                if m:
                    print(f"\n\nFlag:\n{m.group(0).decode()}")
                    return
    finally: r.close()
    print(f"\nNo flag. Collected {len(cts)} lines, keystream bytes {len(ks)}")

if __name__ == "__main__":
    print(f"Streaming from {HOST}:{PORT} …")
    stream(HOST, PORT, TARGET)
```

Output:
```shell
python3 solve4.py  
Streaming from chal.sunshinectf.games:25403 …  
[+] Opening connection to chal.sunshinectf.games on port 25403: Done  
  
  
Flag:  
sun{n3v3r_c0unt_0ut_th3_p1ut0ni4ns}  
[*] Closed connection to chal.sunshinectf.games port 25403
```

`sun{n3v3r_c0unt_0ut_th3_p1ut0ni4ns}`