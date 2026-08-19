---
layout: post
title:  "Crypto Challenges"
date:   2026-08-18 21:37:00 -0400
author: uzi
tags: [CTF]
permalink: /gaslight-ctf-2026-crypto/
---
* TOC
{:toc}

## Where the dream starts 1 

**Category**: crypto

**Author**: william_etotheipi

**Description**: This one's a classic. Can you understand the crypto from the maths?

I get the challenge files:
```python
from string import ascii_lowercase

def encrypt(pt: str, K: int) -> str:
    pt = list(map(lambda x: ascii_lowercase.index(x), list(pt)))
    ct = []
    for x in pt:
        ct.append((x + K) % 26)
    
    ct = list(map(lambda x: ascii_lowercase[x], ct))
    ct = ''.join(ct)
    return ct

with open('gaslight-where-the-dream-starts-1/flag_and_key.txt', 'r') as file:
    flag, K = file.readline().split()
    K = int(K)

print(encrypt(flag, K))
```

```shell
cat output.txt  
fdhvduuhdoobolnhgdvkliwriwkuhh
```

Each character was rotated 23 times. [cyberchef recipe](https://gchq.github.io/CyberChef/#recipe=ROT13(true,true,false,23)&input=ZmRodmR1dWhkb29ib2xuaGdkdmtsaXdyaXdrdWho)

`gaslightCTF{caesarreallylikedashiftofthree}`

___

## Where the dream starts 2

**Category**: crypto

**Author**: william_etotheipi

**Description**: Musicians transpose keys regularly. Cryptanalysts transpose [columns](https://en.wikipedia.org/wiki/Transposition_cipher#Columnar_transposition) regularly.  
  
Keylength: key of cipher in Where the dream starts 1  
Keyword: `ascii_lowercase[:keylength]`

I get the challenge file:
```shell
cat output2.txt  
T aiglhTtn0-t-yf-4}hfgsaitFrss2hk--fteel  sgC{4p3-330gl!o
```

There are 3 columns (keylength). Total length = `57`. Each column has `57/3 = 19` characters. You need to split the ciphertext into 3 equal parts (the columns). Then you read across rows and take the 1st character of each column, then the 2nd, then the 3rd… and join them:
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> ct = "T aiglhTtn0-t-yf-4}hfgsaitFrss2hk--fteel  sgC{4p3-330gl!o"  
... keylen = 3  
... rows = len(ct) // keylen  
...  
... # 3 columns, 19 chars each  
... cols = [ct[i*rows:(i+1)*rows] for i in range(keylen)]  
...  
... pt = ''.join(''.join(cols[c][r] for c in range(keylen)) for r in range(rows))  
... print(pt)  
...  
The flag is gaslightCTF{tr4nsp0s3-2-th3-k3y-0f-g-fl4t!}eo
```

`gaslightCTF{tr4nsp0s3-2-th3-k3y-0f-g-fl4t!}`
