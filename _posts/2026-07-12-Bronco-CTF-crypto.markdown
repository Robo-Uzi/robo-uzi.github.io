---
layout: post
title:  "Crypto Challenges"
date:   2026-07-12 12:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /bronco-CTF-2026-crypto/
---
* TOC
{:toc}

## Grandma's Secret

**Category**: crypto

**Author**: dot.t

**Description**: Grandma wants to protect her wifi password. Can you find out what it is before grandpa does?

I get the challenge file:

![Alt text](/images/Letter.jpeg)

First I need to reverse the columnar transposition (keyword = `SUGAR`), then reverse the Polybius substitution (the 6x6 table). I ran this python: 
```python
#!/usr/bin/env python3
import argparse
import json
import sys

HEADERS = "ADFGVX"


def build_mapping(square_grid):
    if len(square_grid) != 6 or any(len(row) != 6 for row in square_grid):
        raise ValueError("Square must be a 6x6 grid")

    mapping = {}
    for i, row in enumerate(square_grid):
        for j, ch in enumerate(row):
            mapping[HEADERS[i] + HEADERS[j]] = ch
    return mapping


def decrypt_adfgvx(ciphertext, keyword, square_grid):
    # clean input
    ciphertext = "".join(ciphertext.split())
    key = keyword.strip()
    K = len(key)
    N = len(ciphertext)

    if N % K != 0:
        raise ValueError(f"Ciphertext length ({N}) is not divisible by key length ({K})")

    R = N // K

    sorted_indices = sorted(range(K), key=lambda i: (key[i], i))

    chunks = [ciphertext[i * R:(i + 1) * R] for i in range(K)]

    # build the columns of the transposition grid
    columns = [[''] * R for _ in range(K)]
    for col_idx, chunk in zip(sorted_indices, chunks):
        columns[col_idx] = list(chunk)

    # read the grid row by row to recover the raw coordinate symbols
    symbols = ''.join(columns[c][r] for r in range(R) for c in range(K))

    # build the polybius square mapping
    mapping = build_mapping(square_grid)

    # decode pairs of symbols
    plaintext = ''.join(mapping[symbols[i:i+2]] for i in range(0, len(symbols), 2))
    return plaintext


def main():
    parser = argparse.ArgumentParser(description="ADFGVX cipher decoder")
    parser.add_argument("-c", "--ciphertext", required=True,
                        help="Ciphertext string (may contain spaces)")
    parser.add_argument("-k", "--keyword", required=True,
                        help="Transposition keyword")
    parser.add_argument("-s", "--square", required=True,
                        help='Polybius square as a JSON array of 6 strings.')
    args = parser.parse_args()

    try:
        square_grid = json.loads(args.square)
        if not isinstance(square_grid, list) or len(square_grid) != 6:
            raise ValueError("Square must be a JSON array of 6 strings")
    except json.JSONDecodeError:
        print("Error: --square must be a valid JSON array.", file=sys.stderr)
        sys.exit(1)

    try:
        result = decrypt_adfgvx(args.ciphertext, args.keyword, square_grid)
        print(result)
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)


if __name__ == "__main__":
    main()
```

output:
```shell
python3 solve.py -c "GVXXFVXVAFXFXVGADAFF" -k "SUGAR" -s '["B3MRLI","A6F082","C7SEUH","Z9DXKV","1QYW5P","NJT4GO"]'  
JELLYDONUT
```

`bronco{JELLYDONUT}`

___

## Shifting Away

**Category**: crypto

**Author**: yoshie878

**Description**: I'm slowly shifting, shifting afar Char after char, char after char I'm slowly shifting (shifting afar)

And it feels like I'm fighting Underscores against the stream Braces against the stream

(Source Material: Mr. Probz, 2013)

`bqmkyj{Ldfmam_Nfd_Abxjpb_Thhdqeia_Snqn_Vzey_Bok_TdudakQkwfy_Kkhxbte_Yo_Jnfvdeueqq}`

For each character at position `i` (starting at 0), the plaintext letter was shifted backward by `i` places. Example:
- Position 1: `'r'` > shifted backward 1 > `'q'`

Shift each ciphertext letter forward by `i` places to get the original back:
```python
python3  
Python 3.13.5 (main, Jun 13 2026, 14:18:01) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> cipher = "bqmkyj{Ldfmam_Nfd_Abxjpb_Thhdqeia_Snqn_Vzey_Bok_TdudakQkwfy_Kkhxbte_Yo_Jnfvdeueqq}"  
...  
... plain = ""  
... for i, ch in enumerate(cipher):  
...     if 'a' <= ch <= 'z':  
...         plain += chr((ord(ch) - ord('a') + i) % 26 + ord('a'))  
...     elif 'A' <= ch <= 'Z':  
...         plain += chr((ord(ch) - ord('A') + i) % 26 + ord('A'))  
...     else:  
...         plain += ch  
...  
... print(plain)  
...  
bronco{Slowly_But_Surely_Shifting_Away_Into_The_PascalSnake_Strings_Of_Characters}
```

`bronco{Slowly_But_Surely_Shifting_Away_Into_The_PascalSnake_Strings_Of_Characters}`
