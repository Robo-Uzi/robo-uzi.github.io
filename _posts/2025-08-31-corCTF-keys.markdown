---
layout: post
title:  "keys"
date:   2025-08-28 22:25:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /corctf-2025-keys/
---

## keys

**Title:** keys

**Category:** crypto

**Creators:** BrownieInMotion, defund

See attachments:

Downloads:

[encrypt.py](https://static.cor.team/corctf-2025/1d67aee34d3a141d235e74c6195fba8a8c33a0662c5f3d12352a4988d6e27962/encrypt.py)

[flag-enc.bmp](https://static.cor.team/corctf-2025/b44f0e212597a26df7693ef463de2b3024f56c7d524a545059527a5c3a7e9a64/flag-enc.bmp)

```shell
cat encrypt.py  
from functools import reduce  
import itertools  
import os  
  
def xor(x, y):  
   return bytes([a ^ b for a, b in zip(x, y)])  
  
def stream(keys0, keys1):  
   for keys in itertools.product(*zip(keys0, keys1)):  
       key = reduce(xor, keys)  
       yield key  
  
with open('flag.bmp', 'rb') as f:  
   d = f.read()  
   header = d[:len(d) - 1024 ** 2 * 3]  
   data = d[len(d) - 1024 ** 2 * 3:]  
  
keys0 = [os.urandom(3) for _ in range(20)]  
keys1 = [os.urandom(3) for _ in range(20)]  
ks = stream(keys0, keys1)  
  
chunked = itertools.zip_longest(*[iter(data)] * 3)  
encrypted = [  
   xor(chunk, next(ks))  
   for chunk in chunked  
]  
  
with open('flag-enc.bmp', 'wb') as f:  
   f.write(header)  
   f.write(b''.join(encrypted))
```

```shell
file flag-enc.bmp  
flag-enc.bmp: PC bitmap, Windows 98/2000 and newer format, 1024 x 1024 x 24, cbSize 3145866, bits offset 138
```

I am given the image `flag-enc.bmp` to decrypt. 

`encrypt.py` first reads the `bmp` image and splits off the unencrypted header. Then it generates 20 key pairs (40 keys). It creates a key stream `ks = stream(keys0, keys1)` by XORing combinations of the keys. It encrypts the image by XORing each 3 byte chunk with a key from the stream. Finally, it saves the header and the encrypted data as a new bmp. 

The encryption uses all possible XOR combinations of the random keys (creates a key space of 2^20 possible keys for each chunk) while storing 40 keys.

I run this script to decrypt the image:
```python
#!/usr/bin/env python3
import sys, collections

def xor_bytes(a, b):
    # XOR two 3 byte RGB triplets together
    return bytes([a[i] ^ b[i] for i in range(3)])

def read_bmp_pixels(filename):
    data = open(filename, "rb").read()
    # Check for BMP file signature
    if data[:2] != b"BM":
        raise SystemExit("Not a BMP file")
    # Parse BMP header values
    pixel_offset = int.from_bytes(data[10:14], "little")
    width = int.from_bytes(data[18:22], "little")
    height = int.from_bytes(data[22:26], "little")
    bpp = int.from_bytes(data[28:30], "little")
    if bpp != 24:
        raise SystemExit("Expected 24bpp BMP")
    # Each row is padded to a multiple of 4 bytes
    row_size = ((width * 3 + 3) // 4) * 4
    # Extract pixel array
    pixels = data[pixel_offset : pixel_offset + row_size * height]
    if len(pixels) % 3:
        raise SystemExit("Pixel data not divisible by 3")
    # Split pixels into RGB triplets
    triplets = [pixels[i : i + 3] for i in range(0, len(pixels), 3)]
    return data[:pixel_offset], triplets, width, height, row_size

def write_bmp(filename, header, triplets):
    # Write BMP header + pixel data back to file
    open(filename, "wb").write(header + b"".join(triplets))

def guess_bit_key_triplets(cipher_pixels, max_bits=20):
    # Recover per bit keys by analyzing XOR differences across strides
    num_pixels = len(cipher_pixels)
    bit_keys = {}
    for bit_index in range(max_bits):
        stride = 1 << bit_index
        counter = collections.Counter()
        # Compare pixels separated by stride distance
        for n in range(0, num_pixels - stride, 2 * stride):
            diff = xor_bytes(cipher_pixels[n], cipher_pixels[n + stride])
            counter[diff] += 1
        # Choose the most common XOR difference as the key for this bit
        bit_keys[bit_index] = counter.most_common(1)[0][0] if counter else b"\x00\x00\x00"
    return bit_keys

def build_keystream(bit_keys, num_pixels):
    # Construct full keystream by combining bit keys according to pixel index
    ks = [b"\x00\x00\x00"] * num_pixels
    for n in range(num_pixels):
        s = b"\x00\x00\x00"
        # If a given bit in the pixel index is set, include that bits key
        for bit_index in range(20):
            if (n >> bit_index) & 1:
                s = xor_bytes(s, bit_keys[bit_index])
        ks[n] = s
    return ks

def main():
    if len(sys.argv) < 2:
        print("Usage: python3 solve.py flag-enc.bmp")
        return
    # Load encrypted BMP and pixel triplets
    header, cipher_pixels, w, h, row_size = read_bmp_pixels(sys.argv[1])
    num_pixels = len(cipher_pixels)
    assert num_pixels == (1 << 20)  # Expects exactly 2^20 pixels

    # Derive keystream and decrypt pixels
    bit_keys = guess_bit_key_triplets(cipher_pixels)
    keystream = build_keystream(bit_keys, num_pixels)
    step1 = [xor_bytes(c, k) for c, k in zip(cipher_pixels, keystream)]
    # Write partially decrypted image
    write_bmp("step1_decrypted_xorGlobal.bmp", header, step1)

if __name__ == "__main__":
    main()
```
Run it like this `python3 solve.py flag-enc.bmp`. 

I can now see the image. Since it was derived statistically the colors are off but it is readable. 

Original image:

![Alt text](/images/flag-enc.bmp)

Final image:

![Alt text](/images/step1_decrypted_xorGlobal.bmp)

Looks like a slightly rotated image of this fella. 

![Alt text](/images/ctflogo.png)

`corctf{642fcb29f091fb14}`

___
