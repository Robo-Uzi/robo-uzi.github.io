---
layout: post
title:  "jumping"
date:   2025-09-21 10:21:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /K17-2025-jumping/
---

**Title:** jumping

**Category:** beginner rev

**Description:** I made this game in high school and I can't seem to clear it! Can you help? Surely you wouldn't cheat!

I get the file `jumping.zip` which contains this binary:
```shell
file jumping  
jumping: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=f5e4eb9bd95f0a14f41d1ef1a6f8ee703c85a059, stripped
```

If I try to run it I see this:
```shell
./jumping  
pygame 2.6.1 (SDL 2.28.4, Python 3.10.12)  
Hello from the pygame community. https://www.pygame.org/contribute.html  
Traceback (most recent call last):  
 File "test.py", line 11, in <module>  
FileNotFoundError: No file 'man_second.png' found in working directory '/home/user/Desktop/ctf/K17'.  
[PYI-4417:ERROR] Failed to execute script 'test' due to unhandled exception!
```

I do some research and find PyInstaller executables store a compressed archive of `.pyc` files. I can extract them with this script: `https://raw.githubusercontent.com/extremecoders-re/pyinstxtractor/master/pyinstxtractor.py`:
```shell
python3 pyinstxtractor.py jumping  
[+] Processing jumping  
[+] Pyinstaller version: 2.1+  
[+] Python version: 3.10  
[+] Length of package: 47458338 bytes  
[+] Found 294 files in CArchive  
[+] Beginning extraction...please standby  
[+] Possible entry point: pyiboot01_bootstrap.pyc  
[+] Possible entry point: pyi_rth_inspect.pyc  
[+] Possible entry point: pyi_rth_pkgutil.pyc  
[+] Possible entry point: pyi_rth_multiprocessing.pyc  
[+] Possible entry point: pyi_rth_pkgres.pyc  
[+] Possible entry point: pyi_rth_setuptools.pyc  
[+] Possible entry point: pyi_rth_glib.pyc  
[+] Possible entry point: pyi_rth_gio.pyc  
[+] Possible entry point: pyi_rth_gi.pyc  
[+] Possible entry point: pyi_rth_cryptography_openssl.pyc  
[+] Possible entry point: test.pyc  
[!] Warning: This script is running in a different Python version than the one used to build the executable.  
[!] Please run this script in Python 3.10 to prevent extraction errors during unmarshalling  
[!] Skipping pyz extraction  
[+] Successfully extracted pyinstaller archive: jumping  
  
You can now use a python decompiler on the pyc files within the extracted directory
```

I have an issue de compiling the `.pyc` files due to python version or something so I go to [https://www.pylingual.io/](https://www.pylingual.io/view_chimera?identifier=5df0f8cc982f3f5fa8de657745d124026a3dfb57ccf5ac06c84c56aaae4be100). When decompiling `test.pyc` I see this at the bottom:
```python
while not crashed:
	for event in pygame.event.get():
		if event.type == pygame.QUIT:
			crashed = True
			break
	pygame.draw.rect(game_window, light_blue, pygame.Rect((0, 0, 480, 650)))
	somemoretext = bytes((x ^ y for x, y in zip([69, 83, 248, 247, 201, 230, 244, 121, 219, 149, 77, 175, 159, 11, 129, 102, 49, 30, 62, 228, 158, 79, 255, 208, 124, 102, 127, 119, 154, 15, 145, 121, 140, 229, 51, 221, 77, 72, 73, 28, 30, 78, 225, 229, 172, 57, 45, 65, 252, 48], b'\x0eb\xcf\x8c\xf8\xb9\xc0:\xec\xe0y\xe3\xd3r\xde"\x01P\t\xbb\xd5!\xcf\xa7#W9(\xadg\xa5N\xd3\xafF\x90=\x17x)A>\xd1\xd0\x99\x08o-\xb9M'))).decode()
	display_message('You win!')
	display_message(somemoretext, 2)
	pygame.display.flip()
	clock.tick(60)
pygame.quit()
quit()
```

I run this in my terminal and get the flag:
```shell
python3  
Python 3.11.2 (main, Apr 28 2025, 14:11:48) [GCC 12.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> nums = [69, 83, 248, 247, 201, 230, 244, 121, 219, 149, 77, 175, 159, 11, 129, 102, 49, 30, 62, 228, 158, 79, 255, 208, 124, 102, 127, 119, 154, 15, 145, 121, 140, 229, 51, 221, 77, 72, 73, 28, 30, 78, 225, 229, 172, 57, 45, 65, 252, 48]  
>>> bytes_seq = b'\x0eb\xcf\x8c\xf8\xb9\xc0:\xec\xe0y\xe3\xd3r\xde"\x01P\t\xbb\xd5!\xcf\xa7#W9(\xadg\xa5N\xd3\xafF\x90=\x17x)A>\xd1\xd0\x99\x08o-\xb9M'  
>>>    
>>> flag = bytes([x ^ y for x, y in zip(nums, bytes_seq)]).decode()  
>>> print(flag)  
K17{1_4C7u4LLy_D0N7_Kn0w_1F_7h47_JuMp_15_p0551BlE}  
>>> exit()
```

`K17{1_4C7u4LLy_D0N7_Kn0w_1F_7h47_JuMp_15_p0551BlE}`