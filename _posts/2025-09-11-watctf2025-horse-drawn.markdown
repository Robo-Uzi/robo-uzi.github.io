---
layout: post
title:  "horse-drawn"
date:   2025-09-11 18:39:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /watctf-2025-horse-drawn/
---

**Title:** horse-drawn

**Category:** Misc

**Author**: [virchau13](https://github.com/virchau13)

**Description:** This is how it must have felt in the Year of Our Ford.

`ssh hexed@challs.watctf.org -p 8022`

I get this python file:
```python
#!/usr/bin/env python3  
import sys  
assert sys.stdout.isatty()  
flag = open("/flag.txt").read().strip()  
to_print = flag + '\r' + ('lmao no flag for you ' * 32)  
print(to_print)
```
`assert sys.stdout.isatty()` means `stdout` must be a terminal. `\r` moves the cursor back to the start of the same line without advancing down, so `flag + '\r' + ('lmao no flag for you ' * 32)` overwrites the flag text on screen visually.

This is what I get when connecting:
```shell
ssh hexed@challs.watctf.org -p 8022  
The authenticity of host '[challs.watctf.org]:8022 ([172.174.211.227]:8022)' can't be established.  
ED25519 key fingerprint is SHA256:B7txBCIKkrOQM5ayhfL/ahB9HA1hep6ui/xfE77DVY4.  
This key is not known by any other names.  
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes  
Warning: Permanently added '[challs.watctf.org]:8022' (ED25519) to the list of known hosts.  
lmao no flag for you lmao no flag for you lmao no flag for you lmao no flag for you 
lmao no flag for you lmao no flag for you lmao no flag for you lmao no flag for you 
lmao no flag for you lmao no flag for you lmao no flag for you lmao no flag for you 
lmao no flag for you lmao no flag for you lmao no flag for you lmao no flag for you 
lmao no flag for you lmao no flag for you lmao no flag for you lmao no flag for you 
lmao no flag for you lmao no flag for you lmao no flag for you lmao no flag for you 
lmao no flag for you lmao no flag for you lmao no flag for you lmao no flag for you 
lmao no flag for you lmao no flag for you lmao no flag for you lmao no flag for you    
Connection to challs.watctf.org closed.
```

In the raw output the flag still comes before the `\r`. Capture it with `sed -n 'l'` which prints all characters literally. `ssh -tt` forces a psuedo terminal. 
```shell
sshpass -p '' ssh -tt -p 8022 hexed@challs.watctf.org 2>/dev/null | sed -n 'l'  
watctf{im_more_of_a_tram_fan_personally}\rlmao no flag for you lmao n\  
o flag for you lmao no flag for you lmao no flag for you lmao no flag\  
for you lmao no flag for you lmao no flag for you lmao no flag for y\  
ou lmao no flag for you lmao no flag for you lmao no flag for you lma\  
o no flag for you lmao no flag for you lmao no flag for you lmao no f\  
lag for you lmao no flag for you lmao no flag for you lmao no flag fo\  
r you lmao no flag for you lmao no flag for you lmao no flag for you \  
lmao no flag for you lmao no flag for you lmao no flag for you lmao n\  
o flag for you lmao no flag for you lmao no flag for you lmao no flag\  
for you lmao no flag for you lmao no flag for you lmao no flag for y\  
ou lmao no flag for you \r$
```
`watctf{im_more_of_a_tram_fan_personally}`