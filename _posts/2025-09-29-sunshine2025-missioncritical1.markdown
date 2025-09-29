---
layout: post
title:  "Missioncritical1"
date:   2025-09-29 12:46:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /sunshine-2025-missioncritical1/
---
{% raw  %}
**Title:** Missioncritical1

**Category:** rev

**Author:** rivdb

**Description:** Ground Control to Space Cadet!

We've intercepted a satellite control program but can't crack the authentication sequence. The satellite is in an optimal transmission window and ready to accept commands. Your mission: Reverse engineer the binary and find the secret command to gain access to the satellite systems.

I get this binary:
```shell
file chall  
chall: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=643f888f6a846b88e61a61a180506277b41c610d, for GNU/Linux 4.4.0, stripped  

./chall  
Satellite Status: Battery=80%, Orbit=32, Temp=-25C  
Enter satellite command: command  
Access Denied!
```

Open the binary in ghidra. This is the main function:
```d
undefined8 main(void)

{
  int iVar1;
  long in_FS_OFFSET;
  char acStack_98 [64];
  char local_58 [56];
  long local_20;
  
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  sprintf(acStack_98,"sun{%s_%s_%s}\n",&DAT_00102013,"s4t3ll1t3",&DAT_00102004);
  time((time_t *)0x0);
  printf("Satellite Status: Battery=%d%%, Orbit=%d, Temp=%dC\n",0x50,0x20,0xffffffe7);
  printf("Enter satellite command: ");
  fgets(local_58,0x32,stdin);
  iVar1 = strcmp(local_58,acStack_98);
  if (iVar1 == 0) {
    puts("Access Granted!");
  }
  else {
    puts("Access Denied!");
  }
  if (local_20 == *(long *)(in_FS_OFFSET + 0x28)) {
    return 0;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}
```
I can clearly see `"sun{%s_%s_%s}\n",&DAT_00102013,"s4t3ll1t3",&DAT_00102004` so right away I know `sun{A_s4t3ll1t3_B}`.

Look at `.rodata`:
```shell
objdump -s -j .rodata chall  
  
chall:     file format elf64-x86-64  
  
Contents of section .rodata:  
2000 01000200 33313331 00733474 336c6c31  ....3131.s4t3ll1  
2010 74330065 34737900 73756e7b 25735f25  t3.e4sy.sun{%s_%  
2020 735f2573 7d0a0045 6e746572 20736174  s_%s}..Enter sat  
2030 656c6c69 74652063 6f6d6d61 6e643a20  ellite command:    
2040 00416363 65737320 4772616e 74656421  .Access Granted!  
2050 00416363 65737320 44656e69 65642100  .Access Denied!.  
2060 53617465 6c6c6974 65205374 61747573  Satellite Status  
2070 3a204261 74746572 793d2564 25252c20  : Battery=%d%%,    
2080 4f726269 743d2564 2c205465 6d703d25  Orbit=%d, Temp=%  
2090 64430a00                             dC..
```
{% endraw %}
`sun{e4sy_s4t3ll1t3_3131}`