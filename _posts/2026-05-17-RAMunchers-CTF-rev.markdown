---
layout: post
title:  "Reverse Engineering Challenges"
date:   2026-05-17 20:54:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RAMunchers-CTF-2026-rev/
---
* TOC
{:toc}

## Gibson's First

**Author**: not given

**Description**: Early versions of the Gibson left a lot to be desired in terms of generated code quality. Unfortunately, it appears that this code is still being used to secure critical information. Can you hack in and recover this data? [https://ctf.ramunchers.com/engine/challenge/11](https://ctf.ramunchers.com/engine/challenge/11) Port 1337

I get the file `challenge.py`:
```python
#!/usr/bin/python3
import time

def main():
    key = input("Enter the key: ")

    if key[::-1] != "drowssaPIAterceSrepuS":
            return 1

    return 0

if __name__ == "__main__":
    if not main():
        print(f"FLAG: {open('/flag.txt').read()}")
```

To get the key reverse the string `drowssaPIAterceSrepuS`:
```shell
SuperSecretAIPassword
```

Connect and get the flag:
```shell
nc 10.42.65.10 1337  
Enter the key: SuperSecretAIPassword  
FLAG: RMCTF{AI_g3n3r4t3s_BAD_c0de}
```

`RMCTF{AI_g3n3r4t3s_BAD_c0de}`

___

## Authentication Tokens

**Author**: not given

**Description**: We have identified an API that the Gibson's nodes use to communicate with the Gibson itself. So far, none of our team have been able to find a way to generate or crack valid keys to use this API. Can you find a way to generate valid keys and access the API? [https://ctf.ramunchers.com/engine/challenge/12](https://ctf.ramunchers.com/engine/challenge/12). Port 1337

I get the file `challenge.py`:
```python
#!/usr/bin/python3
import time

def main():
    key = input("Enter the key: ")

    if (not key.isalnum()) or (len(key) % 3):
        return 1

    part1 = part2 = part3 = ""

    for i in range(0, len(key), 3):
        part1 += key[i]
        part2 += key[i+2]
        part3 += key[i+1]

    if not (len(part1) >= 8):
        return 1

    target1 = "\x08?'!#!\x1b7"

    for i, keypair in enumerate(zip(part1, "GibsonIs")):
        if chr(ord(keypair[0])^ord(keypair[1])) != target1[i]:
            return 1

    target2 = (int(time.time()) >> 2).to_bytes(8, "little")

    for keypair in zip(part2, target2):
        if chr((keypair[1] % 26) + 0x61) != keypair[0]:
            return 1

    for i, keypair in enumerate(zip(part1, part2)):
        if chr(((ord(keypair[0]) +ord(keypair[1]) )% 26) + 0x61) != part3[i]:
            return 1

    return 0

if __name__ == "__main__":
    if not main():
        print(f"FLAG: {open('/flag.txt').read()}")
```

Reconstructing `part1`: the script performs a XOR operation between `part1` and the string `"GibsonIs"`, then compares it to `target1`.
- Logic: `part1[i] ^ "GibsonIs"[i] == target1[i]`
- Reverse: `part1[i] = target1[i] ^ "GibsonIs"[i]`

```shell
python3  
Python 3.13.5 (main, Apr  6 2026, 12:24:14) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> target1 = "\x08?'!#!\x1b7"  
... part1 = "".join([chr(ord(c) ^ ord(g)) for c, g in zip(target1, "GibsonIs")]) ...  
>>> print(part1)  
OVERLORD  
>>> exit
```

Reconstructing `part2`: the script uses the current system time: `(int(time.time()) >> 2)`. This value updates every 4 seconds.
- Logic: `part2[i] = (target2[i] % 26) + 0x61` (where 0x61 is 'a').
- Calculation:
    1. Get the current unix timestamp
    2. Bit shift right by 2
    3. Convert to 8 bytes (little endian)
    4. For each byte take `mod 26` and add to 'a'

 Reconstructing `part3`: this part is a dependent sum of the first two parts.
- Logic: `part3[i] = ((ord(part1[i]) + ord(part2[i])) % 26) + 0x61`

Solve script:
```python
import time

# part 1
target1 = "\x08?'!#!\x1b7"
part1 = "".join([chr(ord(c) ^ ord(g)) for c, g in zip(target1, "GibsonIs")])

# part 2
t = int(time.time()) >> 2
target2 = t.to_bytes(8, "little")
part2 = "".join([chr((b % 26) + 0x61) for b in target2])

# part 3
part3 = "".join([chr(((ord(part1[i]) + ord(part2[i])) % 26) + 0x61) for i in range(8)])

full_key = []
for i in range(8):
    full_key.append(part1[i])
    full_key.append(part3[i])
    full_key.append(part2[i])

print("".join(full_key))
```

Get the flag:
```shell
python3 solve3.py | nc 10.42.65.10 1337  
Enter the key: FLAG: RMCTF{v3ry_b4d_k3yg3n_5y573m5}
```

`RMCTF{v3ry_b4d_k3yg3n_5y573m5}`
