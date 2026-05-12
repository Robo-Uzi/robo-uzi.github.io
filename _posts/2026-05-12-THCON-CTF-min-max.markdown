---
layout: post
title:  "Min Max (part 1/2)"
date:   2026-05-12 19:34:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THCON-CTF-2026-min-max/
---

**Category**: Crypto

**Author**: Mattéo Ricordeau

**Description**: A debug interface was found on a communications node inside the factory. Encrypted traffic is visible on the bus. Decrypt it to unlock additional features.

`nc 51.103.57.72 4243`

I get the challenge file `servery.py` to see how the server works:
```python
import os
import json
import secrets

FLAG = os.environ["FLAG"]

N = 8


def encrypt_block(K, block):
    n = len(K)
    c = []
    for i in range(n):
        c.append(min(K[i][j] + block[j] for j in range(n)))
    return c


def encrypt_data(K, data):
    while len(data) % N != 0:
        data.append(0)
    ct = []
    for i in range(0, len(data), N):
        ct.append(encrypt_block(K, data[i:i + N]))
    return ct


def main():
    K = [[secrets.randbelow(50) + 1 for _ in range(N)] for _ in range(N)]
    ct = encrypt_data(K, list(FLAG.encode()))

    print("Console locked.")
    print()

    while True:
        print("  [1] Unlock")
        print("  [2] Info")

        try:
            choice = input("\n> ").strip()
        except EOFError:
            break

        if choice == "1":
            try:
                ans = json.loads(input("code> ").strip())
                if encrypt_data(K, ans) == ct:
                    print(FLAG)
                    break
                else:
                    print("Access denied.")
            except Exception:
                print("Invalid input.")

        elif choice == "2":
            print()
            print(f"  Terminal:  B-7")
            print(f"  Last user: vcrypt")
            print(f"  Uptime:    247 days")
            print(f"  Lock mode: custom cipher")
            print(f"  K: {json.dumps(K)}")
            print(f"  ct: {json.dumps(ct)}")
            print()

        else:
            print("Unknown command.")


if __name__ == "__main__":
    main()
```

I connect to the server and get `K` and `ct`:
```shell
nc 51.103.57.72 4243  
SST Dynamics - COMMS-DBG v2.3  
Node: UNKNOWN  
  
[!] Encrypted traffic detected on bus.  
  
1) status  
2) decrypt  
3) rf capture [auth required]  
4) quit  
  
> 1  
  
node:   UNKNOWN  
uptime: 247d 14h  
bus:    encrypted  
K: [[35, 29, 10, 35, 35, 17, 43, 3], [20, 45, 37, 3, 37, 16, 31, 6], [16, 14, 4, 30, 42, 22, 36, 35], [7, 48, 37, 6, 19, 42, 2, 6], [30, 16, 43, 7, 27, 29, 5, 21], [15, 21, 8,12, 10, 23, 40, 47], [39, 41, 19, 8, 9, 5, 13, 22], [27, 10, 43, 50, 2, 23, 8, 28]]  
ct: [[77, 79, 71, 50, 53, 75, 61, 56], [55, 55, 82, 58, 59, 64, 60, 80], [68, 67, 73, 70, 78, 61, 56, 53], [91, 79, 84, 50, 53, 88, 61, 56], [81, 80, 66, 51, 54, 73, 62, 57],[51, 54, 81, 54, 58, 63, 59, 76], [3, 6, 22, 2, 5, 23, 5, 8]]  
  
  1) status  
  2) decrypt  
  3) rf capture [auth required]  
  4) quit  
  
>
```

To decrypt the flag you need to reverse the `encrypt_block` function. The ciphertext `ct` is a list of blocks, each an 8 element list. 

Solve script:
```python
import json

# Data from your terminal output
K = [[35, 29, 10, 35, 35, 17, 43, 3], [20, 45, 37, 3, 37, 16, 31, 6], [16, 14, 4, 30, 42, 22, 36, 35], [7, 48, 37, 6, 19, 42, 2, 6], [30, 16, 43, 7, 27, 29, 5, 21], [15, 21, 8,12, 10, 23, 40, 47], [39, 41, 19, 8, 9, 5, 13, 22], [27, 10, 43, 50, 2, 23, 8, 28]]

ct_blocks = [[77, 79, 71, 50, 53, 75, 61, 56], [55, 55, 82, 58, 59, 64, 60, 80], [68, 67, 73, 70, 78, 61, 56, 53], [91, 79, 84, 50, 53, 88, 61, 56], [81, 80, 66, 51, 54, 73, 62, 57], [51, 54, 81, 54, 58, 63, 59, 76], [3, 6, 22, 2, 5, 23, 5, 8]]

def solve_block(K, ct_block):
    n = len(K)
    p = []
    for j in range(n):
        # The core logic: p_j = max(ct_i - K_ij)
        val = max(ct_block[i] - K[i][j] for i in range(n))
        p.append(val)
    return p

full_ans = []
for block in ct_blocks:
    full_ans.extend(solve_block(K, block))

print(json.dumps(full_ans))
```

Output:
```shell
python3 solve3.py  
[60, 57, 67, 76, 65, 63, 48, 74, 66, 70, 78, 52, 78, 60, 72, 52, 63, 62, 69, 71, 51, 51, 73, 65, 73, 70, 81, 76, 78, 74, 48, 88, 60, 52, 71, 77, 63, 64, 49, 78, 65, 67, 77, 51, 74, 59, 68, 48, 8, 8, 18, 11, 13, 0, 0, 0]
```

Get the flag:
```shell
> 2  
key> [60, 57, 67, 76, 65, 63, 48, 74, 66, 70, 78, 52, 78, 60, 72, 52, 63, 62, 69, 71, 51, 51, 73, 65, 73, 70, 81, 76, 78, 74, 48, 88, 60, 52, 71, 77, 63, 64, 49, 78, 65, 67, 77, 51, 74, 59, 68, 48, 8, 8, 18, 11, 13, 0, 0, 0]  
OK  
THC{fl0yd_w4rsh4ll_m33ts_crypt0gr4phy_1n_th3_tr0p1cs}  
  
1) status  
2) decrypt  
3) rf capture [ready]  
4) quit  
  
>
```

`THC{fl0yd_w4rsh4ll_m33ts_crypt0gr4phy_1n_th3_tr0p1cs}`