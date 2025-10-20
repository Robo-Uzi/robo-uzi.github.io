---
layout: post
title:  "TV_show"
date:   2025-10-20 03:13:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-special-round-tv-show/
---

**Title:** TV_show

**Category:** crypto

**Description:** My Arab friend who's name starts with "S", used to think that mixing a transposition and a substitution makes your ciphertext secure. 

`zCxedke_p_rvlFrizvozuussp{ia__j_rqt}pLa_lemlctmuODtn_btedeva`


flag format: WhiteDCTF{}

**Author:** LordOfAyyubid

To get the flag first I need to vigenere decrypt the ciphertext with key "salah". Then undo a columnar transposition using key "salah". 

To find this I first created a `names.txt` files which contained 145 different arabic names that start with "s". 

I make this script to test each name:
```python
#!/usr/bin/env python3
import sys
from typing import List

def vigenere_decrypt(text: str, key: str) -> str:
    # decrypt using Vigenere
    out, ki = [], 0
    key = key.lower()
    for ch in text:
        if ch.isalpha():
            shift = ord(key[ki % len(key)]) - 97
            base = 97 if ch.islower() else 65
            out.append(chr((ord(ch) - base - shift) % 26 + base))
            ki += 1
        else:
            out.append(ch)
    return "".join(out)

def column_order_from_key(key: str) -> List[int]:
    # return list of column indices in reading order from key
    return [idx for idx, _ in sorted(enumerate(key), key=lambda x: (x[1], x[0]))]

def col_decrypt_by_key(ctext: str, key: str) -> str:
    # undo standard keyed columnar transposition
    ncols, L = len(key), len(ctext)
    nrows = L // ncols + (1 if L % ncols else 0)
    full = L % ncols
    read_order = column_order_from_key(key)
    col_lens = [(nrows if i < full else nrows - 1) if full else nrows for i in range(ncols)]

    cols, idx = [''] * ncols, 0
    for col_idx in read_order:
        l = col_lens[col_idx]
        cols[col_idx] = ctext[idx:idx + l]
        idx += l

    return "".join(cols[c][r] for r in range(nrows) for c in range(ncols) if r < len(cols[c]))

def main():
    if len(sys.argv) != 3:
        print(f"Usage: {sys.argv[0]} <ciphertext> <key>")
        sys.exit(1)

    ciphertext, key = sys.argv[1], sys.argv[2]
    result = col_decrypt_by_key(vigenere_decrypt(ciphertext, key), key)
    print(result)

if __name__ == "__main__":
    main()
```

I run this command to test each name against the python script and only display output which show the correct flag and corresponding name:
```shell
while read name; do  
   result=$(python3 solve4.py "zCxedke_p_rvlFrizvozuussp{ia__j_rqt}pLa_lemlctmuODtn_btedeva" "$name")  
   if echo "$result" | grep -q "White"; then  
       echo "Flag found with name: $name"  
       echo "$result"  
   fi  
done < names.txt  

Flag found with name: salah  
WhiteDCTF{imagine_it_was_used_before_to_secure_informations}  
Flag found with name: saleh  
WhiteDCTF{imageja_et_was_usad_before_to_secqra_infoniapions}
```

`WhiteDCTF{imagine_it_was_used_before_to_secure_informations}`