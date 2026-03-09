---
layout: post
title:  "Deoxyribonucleic acid"
date:   2026-03-09 17:43:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /upCTF-2026-Deoxyribonucleic-acid/
---

**Title**: Deoxyribonucleic acid

**Category**: misc

**Author**: abreu22

**Description**: Returning to fundamental silliness, Goldman et al. (2013)

I get the challenge file:
```shell
ACTCTACGAGTCTACAGAGTCGTCGTATCAGTCTCACGTGAGCGAGTATACAGTGTCGAGCGTGCGACTCGCTACAGAGTCGCTGTAGCACGAGTCTAGTGTGTCGATCGAGTGTAGTCTGTCGTCGTCGCTGTAGCACGAGTATAGTCTGTCGTAGTAGCAGTATGATAGAGCA  
  
#           | 0 | 1 | 2  
#  ---------|---|---|---  
#     A     | C | G | T  
#     C     | G | T | A  
#     G     | T | A | C  
#     T     | A | C | G
```
The table encodes trits (0, 1, 2) as transitions between DNA bases. If the previous base is `A`, trit `0` produces `C`, trit `1` produces `G`, trit `2` produces `T`.

There are 6 trits per byte. 3^6 = 729 > 256 so 6 trits can represent any byte value. Reading the trits in groups of 6 (as big endian base 3 numbers) gives the flag.

Solve script:
```python
seq = ("ACTCTACGAGTCTACAGAGTCGTCGTATCAGTCTCACGTGAGCGAGTATACAGTGTCGAGCGTGCGACTCGCTACAGAGTCGCTGTAGCACGAGTCTAGTGTGTCGATCGAGTGTAGTCTGTCGTCGTCGCTGTAGCACGAGTATAGTCTGTCGTAGTAGCAGTATGATAGAGCA")

#   | 0 | 1 | 2
# A | C | G | T
# C | G | T | A
# G | T | A | C
# T | A | C | G

decode_table = {
    ('A', 'C'): 0,  ('A', 'G'): 1,  ('A', 'T'): 2,
    ('C', 'G'): 0,  ('C', 'T'): 1,  ('C', 'A'): 2,
    ('G', 'T'): 0,  ('G', 'A'): 1,  ('G', 'C'): 2,
    ('T', 'A'): 0,  ('T', 'C'): 1,  ('T', 'G'): 2,
}

trits = [decode_table[(seq[i], seq[i + 1])] for i in range(len(seq) - 1)]
print(f"Trit string: {''.join(map(str, trits))}")

# Group every 6 trits and convert (big endian base 3) to a byte value
# 6 trits can encode any ASCII/byte value
decoded_bytes = []
for i in range(0, len(trits) - 5, 6):
    val = 0
    for j in range(6):
        val = val * 3 + trits[i + j]
    decoded_bytes.append(val)

flag = ''.join(chr(b) for b in decoded_bytes)
print(f"Decoded bytes: {decoded_bytes}")
print(f"Flag: {flag}")
```

Output:
```shell
python3 solve3.py  
Trit string: 011100011011002111010010002121011120002112011002002102010112002201011021002111010212001220011011010202010121011020010112010010010212001220011002010112010001001221002212011122
Decoded bytes: [117, 112, 67, 84, 70, 123, 68, 110, 65, 95, 73, 115, 67, 104, 51, 112, 101, 97, 114, 95, 84, 104, 51, 110, 95, 82, 52, 77, 125]  
Flag: upCTF{DnA_IsCh3pear_Th3n_R4M}
```

`upCTF{DnA_IsCh3pear_Th3n_R4M}`