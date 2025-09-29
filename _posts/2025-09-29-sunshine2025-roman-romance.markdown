---
layout: post
title:  "Roman Romance"
date:   2025-09-29 13:00:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /sunshine-2025-roman-romance/
---

**Title:** Roman Romance

**Category:** rev

**Author:** Uvuv

**Description:** When in Rome...

> currently has nonstandard flag format sunshine{}

I get the file `enc.txt`:
```shell
cat enc.txt  
tvotijof|lO1x`z1v5`s1nAo`iJ6u1sZ~
```

Also a binary. When I run it I get a seg fault:
```shell
file romanromance  
romanromance: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=22691467481039a1263756b806396768fd707772, for GNU/Linux 3.2.0, not stripped

./romanromance  
Segmentation fault
```

I open the binary in ghidra. This is the main function:
```d
undefined8 main(void)

{
  FILE *pFVar1;
  size_t __size;
  void *__ptr;
  undefined8 uVar2;
  size_t sVar3;
  long local_30;
  
  pFVar1 = fopen("flag.txt","r+b");
  fseek(pFVar1,0,2);
  __size = ftell(pFVar1);
  rewind(pFVar1);
  __ptr = malloc(__size);
  if (__ptr == (void *)0x0) {
    fwrite("malloc failed\n",1,0xe,stderr);
    fclose(pFVar1);
    uVar2 = 1;
  }
  else {
    sVar3 = fread(__ptr,1,__size,pFVar1);
    if (sVar3 == __size) {
      for (local_30 = 0; local_30 < (long)__size; local_30 = local_30 + 1) {
        *(char *)((long)__ptr + local_30) = *(char *)((long)__ptr + local_30) + '\x01';
      }
      fclose(pFVar1);
      pFVar1 = fopen("enc.txt","w");
      sVar3 = fwrite(__ptr,1,__size,pFVar1);
      if (sVar3 == __size) {
        free(__ptr);
        fclose(pFVar1);
        puts(&DAT_00102040);
        puts(
            "/*************************************************************************************\ \ \n"
            );
        puts("  MWAHAAHAHAH SAY GOOD-BYTE TO YOUR FLAG ROMAN FILTH!!!!! >:) ");
        puts("  OUR ENCRYPTION METHOD IS TOO STRONG TO BREAK. YOU HAVE TO PAY US >:D ");
        puts(
            "  PAY 18.BTC TO THE ADDRESS 1BEER4MINERSMAKEITRAINCOINSHUNT123 TO GET YOUR FLAG BACK,   "
            );
        puts("  OR WE SACK ROME AND I TAKE HONORIA\'S HAND IN MARRIAGE! SIGNED, ATTILA THE HUN.  \n"
            );
        puts(
            "/*************************************************************************************\ \ \n"
            );
        uVar2 = 0;
      }
      else {
        perror("fwrite");
        free(__ptr);
        fclose(pFVar1);
        uVar2 = 1;
      }
    }
    else {
      perror("fread");
      free(__ptr);
      fclose(pFVar1);
      uVar2 = 1;
    }
  }
  return uVar2;
}
```
The program opens `flag.txt`, finds its size, and reads it into a buffer. It loops over each byte `b` and does `b = b + 1` (the `+ '\x01'` in the decompiled code). It writes that modified buffer to `enc.txt`. I got a seg fault because `flag.txt` did not exist. 

After creating a fake `flag.txt` it runs:
```shell
./romanromance  
⠀⠀⠀⠀⠀⠀⠀⠀⣀⣤⣴⠶⠾⠿⠛⠛⠻⠿⠶⣶⣤⣀⠀⠀⠀⠀⠀⠀⠀⠀  
⠀⠀⠀⠀⠀⠀⢠⣾⠟⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠻⣷⣄⠀⠀⠀⠀⠀⠀  
⠀⠀⠀⠀⠀⢠⡿⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢿⣆⠀⠀⠀⠀⠀  
⠀⠀⠀⠀⠀⣿⠇⡤⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢰⡈⣿⠀⠀⠀⠀⠀  
⠀⠀⠀⠀⠀⣿⡆⣷⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣸⠁⣿⠀⠀⠀⠀⠀  
⠀⠀⠀⠀⠀⠸⣧⢸⡆⢀⣀⣀⣤⡀⠀⠀⢀⣤⣀⣀⡀⠀⡟⣸⡟⠀⠀⠀⠀⠀  
⠀⠀⠀⠀⠀⠀⠹⣿⠁⣿⣿⣿⣿⡟⠀⠀⠸⣿⣿⣿⣿⠆⣿⠟⠀⠀⠀⣀⠀⠀  
⠀⢰⡟⢿⣆⠀⠀⣿⠀⠙⢿⣿⠟⠀⣠⣄⠀⠹⣿⣿⠟⠀⢹⠀⠀⣠⡿⢻⣇⠀  
⣠⡾⠃⠈⠻⢷⣦⣽⣄⡀⠀⠀⠀⢸⣿⣿⣧⠀⠀⠀⢀⣠⣿⣤⡶⠟⠁⠘⢿⣆  
⠻⠷⠶⠶⣶⣤⣈⠙⠻⣿⣷⣦⠀⠸⠋⠙⠟⠀⣠⣾⣿⠟⠋⣁⣠⣴⠶⠶⠾⠟  
⠀⠀⠀⠀⠀⠉⠛⠿⣶⣼⠿⣿⣲⡤⡤⡤⢤⢰⣿⡏⣿⣶⠿⠛⠉⠀⠀⠀⠀⠀  
⠀⠀⠀⠀⠀⠀⢀⣠⣴⣿⡄⠻⣹⡟⡟⡟⣻⣻⠽⠁⣿⣦⣄⡀⠀⠀⠀⠀⠀⠀  
⠀⠀⣶⠾⠶⠾⠟⠋⣁⣼⣷⡀⠀⠉⠉⠉⠉⠀⢀⣼⣧⣀⠉⠛⠷⠶⠿⣶⡄⠀  
⠀⠀⠙⣷⡄⢀⣴⠿⠛⠁⠀⠙⠳⠶⠤⠴⠶⠞⠋⠀⠈⠙⠻⣶⡄⠀⣾⠟⠁⠀  
⠀⠀⠀⢸⣷⡿⠃⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢿⣶⡿⠀⠀⠀  
/*************************************************************************************\    
  
 MWAHAAHAHAH SAY GOOD-BYTE TO YOUR FLAG ROMAN FILTH!!!!! >:)    
 OUR ENCRYPTION METHOD IS TOO STRONG TO BREAK. YOU HAVE TO PAY US >:D    
 PAY 18.BTC TO THE ADDRESS 1BEER4MINERSMAKEITRAINCOINSHUNT123 TO GET YOUR FLAG    
BACK, OR WE SACK ROME AND I TAKE HONORIA'S HAND IN MARRIAGE! SIGNED, ATTILA THE  HUN.
 
/*************************************************************************************\
```

To get the flag just subtract one from each byte of the contents of `enc.txt`:
```python
python3 - <<'PY'  
enc="tvotijof|lO1x`z1v5`s1nAo`iJ6u1sZ~"  
print("".join(chr(ord(c)-1) for c in enc))  
PY  
sunshine{kN0w_y0u4_r0m@n_hI5t0rY}
```

`sunshine{kN0w_y0u4_r0m@n_hI5t0rY}`