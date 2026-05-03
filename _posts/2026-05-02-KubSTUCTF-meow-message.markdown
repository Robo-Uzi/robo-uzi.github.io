---
layout: post
title:  "Meow Message"
date:   2026-05-02 19:17:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /KubSTU-2026-meow-message/
---

**Author**: [@van_1pi](https://t.me/van_1pi)

**Category**: Steganography

**Description**: A text file `message.txt` is given containing an ASCII art cat and a poem in Russian. At first glance — just a cute picture with text. _Hint: "Not everything that seems empty is actually empty."_ Flag format: `KubSTU{...}`

I get the challenge file:
```shell
file message.txt  
message.txt: Unicode text, UTF-8 text  

cat message.txt  
    /\_____/\  
   /  o   o  \  
  ( ==  ^  == )  
   )         (  
  (           )  
 ( (  )   (  ) )  
(__(__)___(__)__)  
  
  *** MEOW! ***  
  
  Мяу-мяу, человек!  
  Я не просто кот,  
  Я хранитель тайн.  
  
  В моих лапках есть  
  секрет один...  
  Но его не видно  
  просто так.  
  Присмотрись! :3
```

Translates to:
```shell
Meow-meow, human! I'm not just a cat, I am the keeper of secrets. In my paws there is There is only one secret... But he is not visible Just like that. Take a closer look! :3
```

The flag is hidden in the whitespace (spaces and tabs):
```shell
xxd message.txt  
00000000: 2020 2020 2f5c 5f5f 5f5f 5f2f 5c20 0920      /\_____/\ .  
00000010: 2009 2009 090a 2020 202f 2020 6f20 2020   . ...   /  o  
00000020: 6f20 205c 2009 0909 2009 2009 0a20 2028  o  \ ... . ..  (  
00000030: 203d 3d20 205e 2020 3d3d 2029 2009 0920   ==  ^  == ) ..  
00000040: 2020 0920 0a20 2020 2920 2020 2020 2020    . .   )  
00000050: 2020 2820 0920 0920 2009 090a 2020 2820    ( . .  ...  (  
00000060: 2020 2020 2020 2020 2020 2920 0920 0920            ) . .  
00000070: 0920 200a 2028 2028 2020 2920 2020 2820  .  . ( (  )   (  
00000080: 2029 2029 2009 2009 2009 2009 0a28 5f5f   ) ) . . . ..(__  
00000090: 285f 5f29 5f5f 5f28 5f5f 295f 5f29 2009  (__)___(__)__) .  
000000a0: 0909 0920 0909 0a20 0909 0920 0909 090a  ... ... ... ....  
000000b0: 2020 2a2a 2a20 4d45 4f57 2120 2a2a 2a20    *** MEOW! ***  
000000c0: 0909 2009 2020 200a 2020 0909 2020 2009  .. .   .  ..   .  
000000d0: 0a20 20d0 9cd1 8fd1 832d d0bc d18f d183  .  ......-......  
000000e0: 2c20 d187 d0b5 d0bb d0be d0b2 d0b5 d0ba  , ..............  
000000f0: 2120 0909 0920 0920 200a 2020 d0af 20d0  ! ... .  .  .. .  
00000100: bdd0 b520 d0bf d180 d0be d181 d182 d0be  ... ............  
00000110: 20d0 bad0 bed1 822c 2020 0909 2020 0909   ......,  ..  ..  
00000120: 0a20 20d0 af20 d185 d180 d0b0 d0bd d0b8  .  .. ..........  
00000130: d182 d0b5 d0bb d18c 20d1 82d0 b0d0 b9d0  ........ .......  
00000140: bd2e 2009 2009 0909 0909 0a20 0909 0920  .. . ...... ...  
00000150: 2009 090a 2020 d092 20d0 bcd0 bed0 b8d1   ...  .. .......  
00000160: 8520 d0bb d0b0 d0bf d0ba d0b0 d185 20d0  . ............ .  
00000170: b5d1 81d1 82d1 8c20 0909 0920 2020 200a  ....... ...    .  
00000180: 2020 d181 d0b5 d0ba d180 d0b5 d182 20d0    ............ .  
00000190: bed0 b4d0 b8d0 bd2e 2e2e 2020 0909 2009  ..........  .. .  
000001a0: 2020 0a20 20d0 9dd0 be20 d0b5 d0b3 d0be    .  .... ......  
000001b0: 20d0 bdd0 b520 d0b2 d0b8 d0b4 d0bd d0be   .... ..........  
000001c0: 2009 0920 2020 0909 0a20 20d0 bfd1 80d0   ..   ...  .....  
000001d0: bed1 81d1 82d0 be20 d182 d0b0 d0ba 2e20  ....... .......  
000001e0: 2009 0920 2009 090a 2020 d09f d180 d0b8   ..  ...  ......  
000001f0: d181 d0bc d0be d182 d180 d0b8 d181 d18c  ................  
00000200: 2120 3a33 2009 0909 0909 2009 0a         ! :3 ..... ..
```

This python will read the file as text, then split it into lines. Then it counts how many spaces/tabs are at the end of the line and concatenates them. Each space equals `0` and each tab equals `1`:
```python
data = open("message.txt","r",encoding="utf-8",errors="ignore").read()  
  
bits = ""  
for line in data.splitlines():  
	trail = len(line) - len(line.rstrip(" \t"))  
	if trail:  
		ws = line[-trail:]  
		for c in ws:  
			bits += "0" if c == " " else "1"  
  
print(bits) 
```

```shell
python3 solve.py  
01001011011101010110001001010011010101000101010101111011011101110110100000110001011101000011001101011111011100110111000000110100011000110011001101111101
```

Decode from binary on cyberchef to get the flag.

`KubSTU{wh1t3_sp4c3}`