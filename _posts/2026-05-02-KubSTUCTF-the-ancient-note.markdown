---
layout: post
title:  "The Ancient Note"
date:   2026-05-02 19:17:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /KubSTU-2026-the-ancient-note/
---

**Author**: [@van_1pi](https://t.me/van_1pi)

**Category**: Steganography

**Description**: We are given a text file ancient_note.txt — supposedly an ancient manuscript from an abandoned library. The text is in English, philosophical reflections on the search for hidden truth.

I get the challenge file:
```shell
file ancient_note.txt  
ancient_note.txt: Unicode text, UTF-8 text, with CRLF line terminators  

cat ancient_note.txt  
T​h‌e​ ​M‌a​n‌u‌s​c‌r‌i‌p​t‌ ​o‌f​ ‌S‌h​a​d​o‌w​s​  
  
I‌n​ ‌t​h​e‌ ‌q​u‌i​e‌t​ ‌h​a​l​l‌s​ ‌o​f‌ ​t‌h​e‌ ‌f‌o‌r​g‌o‌t​t‌e‌n​ ‌l​i​b​r​a​r‌y‌,​ ​w​h‌e​r‌e‌ ​d​u‌s​t​ ​s‌e‌t​t​l‌e​s​ ​u​p‌o‌n​ ​a‌n‌c​i‌e‌n​t‌ ‌t‌ome‌s​ a‌n‌d‌ ‌m‌o​o‌n‌l‌i​g‌h​t​ ​f‌i‌l‌t​e​r‌s​ ​t‌h‌r‌o​u‌g​h‌ ​c‌r‌a‌c​k‌e​d​ ​w‌i‌n​d‌o​w​s​,​ ‌t​h‌e‌r‌e‌ ‌e​x‌i‌s​t​s​ ‌a​ ​p​e‌c‌u​l​i‌a‌r​ ‌d‌o‌cu‌m​e​n​t‌.‌ W‌r​i‌t‌t‌e​n​ ‌b‌y​ ​a‌n‌ ​u​n‌k‌n​o​w‌n‌ ​s‌c‌h​o‌l‌a‌r​ ​c‌e‌n‌t‌u‌r​i‌es ago, it speaks of hidden knowledge.  
  
"Tо seek the truth, one must look beyond what the eyes pеrceive.  
The answers lie not in the оbvious, but in the spacеs between.  
Еvery symbol carries weight, yet some carry more than others.  
Wise men know that reality often wеars a mask of normalcy.  
Іn plain sight, secrets dance like shadows at noon.  
Doubt everything you see, for appearancе deceives.  
The hidden shall reveal itself to those who truly оbserve.  
Hіdden paths await those who question the surface."  
  
Many have tried to decipher this text, believing it holds the location  
of a great treasure. Some say it points to knowledge beyond measure.  
Others dismiss it as the ramblings of a madman lost to time.  
  
Yet the manuscript remains, waiting for one who can see past the veil.  
Perhaps you are the one destined to uncover what lies beneath?  
  
- Found in the Archives of the Unnamed Library, circa 1847
```

The text has zero width space and zero width non-joiner unicode between the letters:
```shell
xxd ancient_note.txt | head  
00000000: 54e2 808b 68e2 808c 65e2 808b 20e2 808b  T...h...e... ...  
00000010: 4de2 808c 61e2 808b 6ee2 808c 75e2 808c  M...a...n...u...  
00000020: 73e2 808b 63e2 808c 72e2 808c 69e2 808c  s...c...r...i...  
00000030: 70e2 808b 74e2 808c 20e2 808b 6fe2 808c  p...t... ...o...  
00000040: 66e2 808b 20e2 808c 53e2 808c 68e2 808b  f... ...S...h...  
00000050: 61e2 808b 64e2 808b 6fe2 808c 77e2 808b  a...d...o...w...  
00000060: 73e2 808b 0d0a 0d0a 49e2 808c 6ee2 808b  s.......I...n...  
00000070: 20e2 808c 74e2 808b 68e2 808b 65e2 808c   ...t...h...e...  
00000080: 20e2 808c 71e2 808b 75e2 808c 69e2 808b   ...q...u...i...  
00000090: 65e2 808c 74e2 808b 20e2 808c 68e2 808b  e...t... ...h...
```

I double check by going to [https://www.babelstone.co.uk/Unicode/whatisit.html](https://www.babelstone.co.uk/Unicode/whatisit.html) and inputting the text:
```shell
U+0054 : LATIN CAPITAL LETTER T
U+200B : ZERO WIDTH SPACE [ZWSP]
U+0068 : LATIN SMALL LETTER H
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+0065 : LATIN SMALL LETTER E
U+200B : ZERO WIDTH SPACE [ZWSP]
U+0020 : SPACE [SP]
U+200B : ZERO WIDTH SPACE [ZWSP]
U+004D : LATIN CAPITAL LETTER M
U+200C : ZERO WIDTH NON-JOINER [ZWNJ]
U+0061 : LATIN SMALL LETTER A
U+200B : ZERO WIDTH SPACE [ZWSP]
... etc
```

I run this python to get the binary:
```python
import sys

mapping = {'\u200b': '0', '\u200c': '1'}
text = open(sys.argv[1], encoding='utf-8').read() if len(sys.argv) > 1 else sys.stdin.read()
sys.stdout.write(''.join(mapping[ch] for ch in text if ch in mapping))
```

Output:
```shell
python3 solve2.py ancient_note.txt  
01001011011101010110001001010011010101000101010101111011011010000011000101100100011001000011001101101110010111110111010001110010011101010111010001101000010111110110001000110011011101000111011100110011001100110110111001111101
```

Decode from binary on cyberchef to get the flag.

`KubSTU{h1dd3n_truth_b3tw33n}`