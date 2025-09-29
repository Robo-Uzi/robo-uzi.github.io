---
layout: post
title:  "the poop challenge"
date:   2025-09-29 12:57:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /sunshine-2025-the-poop-challenge/
---

**Title:** the poop challenge

**Category:** misc

**Author:** no one brave enough to claim this one 

**Description:** its da poop challenge :DDDDDDDDDD

I get a text file containing this:
```
💩💩​💩​💩​💩💩💩​💩​
💩💩​💩​💩​💩💩​💩💩​
💩💩​💩​💩💩​💩​💩​💩
💩💩​💩​💩​💩​💩💩​💩​
💩💩​💩​💩💩​💩​💩💩
💩💩​💩​💩💩💩​💩💩​
💩💩​💩​💩​💩💩💩​💩​
💩💩​💩​💩​💩💩💩​💩​
💩💩​💩​💩​💩💩💩​💩​
💩💩​💩​💩💩💩​💩​💩​
💩💩​💩​💩💩​💩​💩​💩​
💩💩​💩​💩💩​💩​💩​💩​
💩💩​💩​💩💩​💩​💩​💩​
💩💩​💩💩​💩​💩​💩​💩​
💩💩​💩​💩​💩💩💩​💩​
💩💩​💩​💩💩​💩​💩​💩​
💩💩​💩​💩💩​💩​💩💩
💩💩​💩​💩​💩💩​💩​💩
💩💩​💩​💩💩💩​💩💩​
💩💩​💩​💩💩💩​💩💩
💩💩​💩💩​💩​💩​💩​💩​
💩💩​💩​💩​💩💩​💩💩
💩💩​💩​💩💩​💩💩💩
💩💩​💩​💩💩💩​💩💩​
💩💩​💩💩​💩​💩​💩​💩​
💩💩​💩​💩​💩💩💩💩
💩💩​💩​💩💩​💩​💩​💩​
💩💩​💩​💩💩​💩​💩​💩​
💩💩​💩​💩​💩💩💩💩
💩💩​💩💩​💩​💩​💩​💩​
💩💩​💩​💩💩💩💩​💩​
💩💩​💩​💩💩​💩💩💩
💩💩​💩​💩💩💩💩💩​
💩💩​💩​💩💩​💩​💩💩
💩💩​💩​💩💩​💩​💩💩
💩💩​💩​💩💩💩​💩💩​
💩💩​💩​💩💩​💩​💩​💩
💩💩​💩​💩💩💩​💩​💩​
💩💩​💩​💩💩💩​💩💩​
💩💩💩​💩💩💩💩💩​
💩💩​💩​💩​💩​💩​💩💩​
```

I put the emojis into [https://www.babelstone.co.uk/Unicode/whatisit.html](https://www.babelstone.co.uk/Unicode/whatisit.html) and I see they are using `U+1F4A9 : PILE OF POO {dog dirt}` along with `U+200B : ZERO WIDTH SPACE [ZWSP]`. 

An emoji followed by a zero width space equals 1. An emoji alone equals 0. I run this script to get the flag:
```python
#!/usr/bin/env python3
import sys

ZWSP = '\u200b'
POO = '\U0001f4a9'

def extract_bits(text: str) -> str:
    return ''.join(
        '1' if i + 1 < len(text) and text[i + 1] == ZWSP else '0'
        for i, c in enumerate(text) if c == POO
    )

def bits_to_bytes(bitstring: str, group: int = 8, msb: bool = True) -> bytes:
    b = bytearray()
    for i in range(0, len(bitstring) - len(bitstring) % group, group):
        piece = bitstring[i:i + group]
        if not msb:
            piece = piece[::-1]
        b.append(int(piece, 2))
    return bytes(b)

def main():
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <file>")
        sys.exit(1)

    with open(sys.argv[1], encoding='utf-8', errors='replace') as f:
        text = f.read()

    bits = extract_bits(text)
    print(f"Total bits found: {len(bits)}")

    decoded_bytes = bits_to_bytes(bits)
    decoded_text = decoded_bytes.decode('utf-8', errors='replace')

    print(f"Decoded text:\n{decoded_text}")

if __name__ == '__main__':
    main()
```

Run the script:
```shell
python3 solve.py poop_challenge.txt  
Total bits found: 328  
Decoded text:  
sun{lesssgooo_solved_the_poop_challenge!}
```

`sun{lesssgooo_solved_the_poop_challenge!}`