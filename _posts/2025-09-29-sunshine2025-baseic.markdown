---
layout: post
title:  "BASEic"
date:   2025-09-29 12:43:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /sunshine-2025-baseic/
---

**Title:** BASEic

**Category:** rev

**Author:** Tyler (brosu)

**Description:** The space base is in danger and we lost the key to get in!

I get this binary:
```shell
file BASEic  
BASEic: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=4c27a8522e5cde996407511f18c475d48dda4ea9, for GNU/Linux 3.2.0, stripped  

./BASEic  
What is the flag> sun{this-is-it}  
You don't get the flag that easily
```

I open it in ghidra. I find the main function which looks like this:
```d
undefined8 main(void)

{
  int comparison;
  size_t sVar1;
  char *__s1;
  long in_FS_OFFSET;
  undefined8 constant_1;
  undefined4 constant_2;
  undefined2 constant_3;
  char local_48 [56];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  constant_1 = 0x4d544e3049305879;
  constant_2 = 0x3d516631;
  constant_3 = 0x3d;
  printf("What is the flag> ");
  __isoc99_scanf(&DAT_00102094,local_48);
  sVar1 = strlen(local_48);
  if (sVar1 == 0x16) {
    sVar1 = strlen(local_48);
    __s1 = (char *)FUN_001012c6(local_48,sVar1);
    comparison = strncmp(__s1,"c3Vue2MwdjNyMW5nX3V",0x13);
    if (comparison == 0) {
      sVar1 = strlen((char *)&constant_1);
      comparison = strncmp(__s1 + 0x13,(char *)&constant_1,sVar1);
      if (comparison == 0) {
        puts("You got it, submit the flag!");
      }
      else {
        puts("Soo Close");
      }
    }
    else {
      puts("Closer");
    }
    free(__s1);
  }
  else {
    puts("You don\'t get the flag that easily");
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```
The program checks for a 22 character input, then calls `FUN_001012c6` which base64 encodes the input. It compares the result to the string `c3Vue2MwdjNyMW5nX3V + yX0I0NTM1fQ==` so the full string is `c3Vue2MwdjNyMW5nX3VyX0I0NTM1fQ==`. `yX0I0NTM1fQ==` comes from the 3 constants. 

I decode this to get the flag:
```shell
echo -n 'c3Vue2MwdjNyMW5nX3VyX0I0NTM1fQ==' | base64 -d  
sun{c0v3r1ng_ur_B4535}
```

`sun{c0v3r1ng_ur_B4535}`