---
layout: post
title:  "Misc Challenges"
date:   2026-02-22 18:05:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /BITSCTF-2026-misc-challenges/
---
* TOC
{:toc}

## Sanity Check 2

**Author**: Aur0r4

**Category**: misc

**Description**: Wait, did you not read the rules ?

In the discord rules its says this:
```shell
BITSCTF 2026: Rules & Guidelines

General Rules
Team Size: There is no team size.
Flag Format: Unless specified otherwise, all flags follow the format ​​​‭‭​​​‍‎‬​​​‌‍‬​​​‌‬‬​​​‌⁣​‭‭​​​‌⁢﻿​​​‌‍⁢​​​‌⁣​​​​‌‬​​​​‍‬⁢BITSCTF​​​‌‍‬​​​‌‬‬​​​‌⁣​​​​‌⁢﻿​​​‌‍⁢{​​​‌⁣​​​​‌‬​​​​‍‬⁢​​​‍‌⁢​​​‌​‬​​​‍‌‍​​​‌​‍​​​‌﻿⁢​​​‌​⁢​​​‍‍⁣​​​‍‍‍​​​‌​‍​​​‌﻿⁢​​​‍‬‍​​​​﻿﻿your_​​​‍‍⁣​​​‌﻿⁢​​​‍‍‍​​​‌​‍​​​‌​‬​​​‍​‍​​​‌﻿⁢​​​‌​‍​​​‍‍﻿​​​‌​‍​​​‍‍‍​​​‍‬‍​​​‌​﻿​​​‍​﻿​​​‌​​​​​‍‌⁣​​​‍​⁣​​​‌﻿⁢​​​‍​‌flag​​​‌​‬​​​‍‍‍​​​‌​‍​​​‍​⁢​​​‍‍⁣​​​‍‌‬​​​‍‌‬​​​‍‬‍​​​‍‬﻿_goes_here}.

Prohibited Actions 
No Flag Sharing: Do not share flags, solutions, or challenge write-ups during the competition.

Communication Guidelines

Use Discord for Queries: All clarifications and discussions should happen in the official BITSCTF Discord server.
No Unsolicited DMs/Pings: Avoid direct messaging admins or spamming challenge authors.
Respect & Courtesy: Treat fellow participants, organizers, and admins with respect. Any form of abuse, harassment, or unsportsmanlike behavior will not be tolerated.

Technical Rules
No Malicious Activity: Do not attack or tamper with competition infrastructure.
No Unauthorized Scanning: Running automated scanners (e.g., Burp Suite Intruder, SQLmap) is strictly prohibited, unless explicitly allowed in a challenge.

Event Schedule
The competition runs from 20-02-2026 (11:00 IST) → 22-02-2026 (11:00 IST).

Failure to follow these rules may result in penalties or disqualification at the discretion of the organizers.

Stay ethical, stay competitive, and most importantly; have fun hacking!
```

The flag is encoded between `your` and `flag` in the example flag: `your_​​​‍‍⁣​​​‌﻿⁢​​​‍‍‍​​​‌​‍​​​‌​‬​​​‍​‍​​​‌﻿⁢​​​‌​‍​​​‍‍﻿​​​‌​‍​​​‍‍‍​​​‍‬‍​​​‌​﻿​​​‍​﻿​​​‌​​​​​‍‌⁣​​​‍​⁣​​​‌﻿⁢​​​‍​‌flag` 

By going to [https://www.babelstone.co.uk/Unicode/whatisit.html](https://www.babelstone.co.uk/Unicode/whatisit.html) I can see unicode like this:
```shell
U+0079 : LATIN SMALL LETTER Y
U+006F : LATIN SMALL LETTER O
U+0075 : LATIN SMALL LETTER U
U+0072 : LATIN SMALL LETTER R
U+005F : LOW LINE
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200D : ZERO WIDTH JOINER [ZWJ]
U+200D : ZERO WIDTH JOINER [ZWJ]
U+2063 : INVISIBLE SEPARATOR {invisible comma}
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+FEFF : ZERO WIDTH NO-BREAK SPACE [ZWNBSP] (alias BYTE ORDER MARK [BOM]) {BOM, ZWNBSP}
U+2062 : INVISIBLE TIMES
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200D : ZERO WIDTH JOINER [ZWJ]

... etc etc ...

U+200D : ZERO WIDTH JOINER [ZWJ]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+2063 : INVISIBLE SEPARATOR {invisible comma}
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200D : ZERO WIDTH JOINER [ZWJ]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+2063 : INVISIBLE SEPARATOR {invisible comma}
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+FEFF : ZERO WIDTH NO-BREAK SPACE [ZWNBSP] (alias BYTE ORDER MARK [BOM]) {BOM, ZWNBSP}
U+2062 : INVISIBLE TIMES
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200D : ZERO WIDTH JOINER [ZWJ]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+0066 : LATIN SMALL LETTER F
U+006C : LATIN SMALL LETTER L
U+0061 : LATIN SMALL LETTER A
U+0067 : LATIN SMALL LETTER G
```

They use 7 different types of unicode so I assume it decodes from base 7.

I run this python to get the flag:
```python
#!/usr/bin/env python3

encoded = "​‌⁣​​​​‌‬​​​​‍‬⁢​​​‍‌⁢​​​‌​‬​​​‍‌‍​​​‌​‍​​​‌﻿⁢​​​‌​⁢​​​‍‍⁣​​​‍‍‍​​​‌​‍​​​‌﻿⁢​​​‍‬‍​​​​﻿﻿your_​​​‍‍⁣​​​‌﻿⁢​​​‍‍‍​​​‌​‍​​​‌​‬​​​‍​‍​​​‌﻿⁢​​​‌​‍​​​‍‍﻿​​​‌​‍​​​‍‍‍​​​‍‬‍​​​‌​﻿​​​‍​﻿​​​‌​​​​​‍‌⁣​​​‍​⁣​​​‌﻿⁢​​​‍​‌flag​​​‌​‬​​​‍‍‍​​​‌​‍​​​‍​⁢​​​‍‍⁣​​​‍‌‬​​​‍‌‬​​​‍‬‍​​​‍‬﻿"

# Base 7 mapping
MAP = {
    "\u200b": "0",  # ZWSP
    "\u200c": "1",  # ZWNJ
    "\u200d": "2",  # ZWJ
    "\u202c": "3",  # PDF
    "\u2062": "4",  # Invisible Times
    "\u2063": "5",  # Invisible Separator
    "\ufeff": "6",  # BOM
}

# Extract base 7 digit stream
digits = "".join(MAP[c] for c in encoded if c in MAP)

# Split on delimiter (000) and decode
flag = "".join(
    chr(int(seg, 7))
    for seg in digits.split("000")
    if seg and 32 <= int(seg, 7) <= 126
)

print("BITSCTF" + flag)
```

Output:
```shell
python3 solve1.py  
BITSCTF{m4k3_5ur3_y0u_r34d_3v3ry7hng_c4r3fully}
```

`BITSCTF{m4k3_5ur3_y0u_r34d_3v3ry7hng_c4r3fully}`

For some reason the flag printed incorrectly!!! I guessed it by adding a `1` by noticing that `3v3ry7hng` is spelled wrong. By manually fixing that, I get the flag!

`BITSCTF{m4k3_5ur3_y0u_r34d_3v3ry7h1ng_c4r3fully}`

___

## Radio Telescope

**Author**: house

**Category**: misc

**Description**: From: Dr. Xavier, To: Chief Officer of Operations, RT-7.

I suspect there are problems with the radio telescope. I've it aimed at the Vega cluster for the better part of the day now, and for the most part its what you expect: noise, a whole lot of it. However the feed of the telescope seems to randomly skip a beat: a clearing in a forest, so to speak. Interesting to say the least. I'm clocking out for the day, and I hope you'll take the telescope offline and put it up for maintenance, and perhaps fix the problem. I know you'll have to justify the costs to the Agency, and so attached is the log of the telescope.

Ciao.

I get the challenge file `rt7-log.txt`:
```shell
cat rt7-log.txt | wc -l  
10000  

cat rt7-log.txt | head  
1.227758032072523804e+02  
1.330683101555940198e+02  
-4.273338156528737386e+00  
1.202651081886457689e+02  
1.329528445634176705e+02  
-2.304570842118583585e+01  
5.462256379619100244e+01  
8.287355707266596028e+01  
1.440850843552550486e+01  
1.074915967105357879e+02  

cat rt7-log.txt | tail  
4.013908621931924614e+01  
1.248962146904023598e+02  
3.030068879890678346e+01  
1.154211383386738703e+02  
-8.151771077071600757e+00  
1.809810553577515861e+02  
5.201510203023995871e+01  
5.294007787562520662e+00  
3.096768509049896068e+01  
5.039587801409842172e+01
```
There are 10,000 floating point numbers in scientific notation.

Solve script:
```python
import numpy as np

with open('rt7-log.txt') as f:
    data = [float(line) for line in f]

threshold = 0.5
min_len = 5
regions = []
i = 0
while i < len(data):
    j = i
    while j < len(data) and np.std(data[i:j+1]) < threshold:
        j += 1
    if j - i >= min_len:
        regions.append(np.mean(data[i:j]))
        i = j
    else:
        i += 1

flag = ''.join(chr(int(round(r))) for r in regions)
print(flag)
```
The script looks for sequences where the numbers are nearly constant (5 numbers or more) and takes the average of all numbers in the constant region (rounding to the nearest integer). Convert that integer to an ASCII character and print the flag.

Output:
```shell
python3 solve2.py  
CTF{s1l3nc3_1n_th3_n01s3}
```
I need to put `BITS` in the flag and then it is correct!

`BITSCTF{s1l3nc3_1n_th3_n01s3}`
