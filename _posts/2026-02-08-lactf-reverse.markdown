---
layout: post
title:  "Reverse Challenges"
date:   2026-02-08 23:19:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /lactf-2026-reverse-challenges/
---
* TOC
{:toc}

## ooo

**Category**: rev

**Author**: k

**Description**: Surely everyone knows the difference between the Cyrillic small letter o, the Greek small letter omicron, and the Latin small letter o, right? :)

I get the challenge file `ooo.py`:
```python
def о(a, b):  
    return a+b  
def ο(a, b):  
    return a-b  
def օ(a, b):  
    return a*b  
def ỏ(a, b):  
    return a//b  
def ơ(a, b):  
    return a^b  
def ó(a, b):  
    return a|b  
def ὀ(a, b):  
    return a&b  
def ὸ(a, b):  
    return b-a  
def ὄ(a, b):  
    return a  
def ὂ(a, b):  
    return b  
def ȯ(a, b):  
    return a % b  
      
  
ὁ = [205, 196, 215, 218, 225, 226, 1189, 2045, 2372, 9300, 8304, 660, 8243, 16057, 16113, 16057, 16004, 16007, 16006, 8561, 805, 346, 195, 201, 154, 146, 223]  
  
guess = input("What's the flag? ") # remember, flags start with lactf{  
  
if (len(guess) < len(ὁ)):  
    print("That's too short :(")  
    exit()  
      
for ö in range(len(ὁ)-1):  
    ό = ord(guess[ö])  
    ὃ = ord(guess[ö+1])  
    if (о(ὄ(ό,ὃ),ὂ(ό,ὃ)) != ὁ[ơ(ö,ȯ(օ(ό,ὃ),ό))]):  
        print("That's not the flag :(")  
        exit()  
      
print("That's the flag! :)")
```
The script uses unicode homoglyphs and obfuscation to make a linear equation. 

Solve script:
```python
vals = [205, 196, 215, 218, 225, 226, 1189, 2045, 2372, 9300, 8304, 660,  
        8243, 16057, 16113, 16057, 16004, 16007, 16006, 8561, 805, 346,  
        195, 201, 154, 146, 223]  
  
flag = list("lactf{")  
  
for i in range(len(flag)-1, len(vals)):  
    prev = ord(flag[i])  
    nxt = vals[i] - prev  
    flag.append(chr(nxt))  
  
print("".join(flag))
```

`lactf{gоοօỏơóὀόὸὁὃὄὂȯöd_j0b}`

___
