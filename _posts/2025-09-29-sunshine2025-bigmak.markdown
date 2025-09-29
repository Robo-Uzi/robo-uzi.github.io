---
layout: post
title:  "BigMak"
date:   2025-09-29 12:34:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /sunshine-2025-bigmak/
---

**Title:** BigMak

**Category:** misc

**Author:** oatzs

**Description:** I tried typing out the flag for you, but our Astronaut Coleson seems to have changed the terminal's keyboard layout? He went out to get a big mak so I guess we're screwed. Whatever, here's the flag, if you can somehow get it back to normal.

`rlk{blpdfp_iajylg_iyi}`

It is based on the `Colemak` keyboard layout [https://en.wikipedia.org/wiki/Colemak](https://en.wikipedia.org/wiki/Colemak). Convert the colemak output to normal qwerty mapping. 

Run this python:
```python
python3  
Python 3.11.2 (main, Apr 28 2025, 14:11:48) [GCC 12.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> colemak_to_qwerty = {  
... 'q':'q','w':'w','f':'e','p':'r','g':'t','j':'y','l':'u','u':'i','y':'o',  
... 'a':'a','r':'s','s':'d','t':'f','d':'g','h':'h','n':'j','e':'k','i':'l',  
... 'z':'z','x':'x','c':'c','v':'v','b':'b','k':'n','m':'m'  
... }  
>>>
>>> def decode(s):  
... return ''.join(colemak_to_qwerty.get(ch, ch) for ch in s)  
... 
>>> print(decode("rlk{blpdfp_iajylg_iyi}"))  
sun{burger_layout_lol}  
>>> exit()
```

`sun{burger_layout_lol}`