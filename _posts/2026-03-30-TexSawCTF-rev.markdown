---
layout: post
title:  "Rev Challenges"
date:   2026-03-30 20:37:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /TexSawCTF-2026-rev/
---
* TOC
{:toc}

## Broken Quest

**Author**: Written by Wulfyrn

**Category**: rev

**Description**: These quest flags are totally just broken! I can't figure out how to complete the quest. Note: There's two ways to do this. Flag format: `texsaw{example_flag}`

I get the challenge files:
```shell
file *  
brokenquest: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=fa0b9ceddd249a3f1d3e401443b9986066c396fd, for GNU/Linux 3.2.0, not stripped  
libc.so.6:   ELF 64-bit LSB shared object, x86-64, version 1 (GNU/Linux), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=274eec488d230825a136fa9c4d85370fed7a0a5e, for GNU/Linux 3.2.0, stripped

./brokenquest  
Interact with objects to advance your quest. Set all quest objective flags and turn in the quest to get the real flag.  
Current Values: [ 0     0       0       0       0       0       0       0       ]  
Choose an action [0-8] to (probably) advance your quest:  
0: Turn in Quest  
1: Reset  
2: Rotate Pillars  
3: Increase Heat  
4: Move Gold Coins  
5: Swing Sword  
6: Swap Gems  
7: Shift Sand Piles  
8: Reverse Polarity  
  
```

Opening the binary in Ghidra, I find these functions:

`main()`:
```shell
undefined8 main(void)

{
  long in_FS_OFFSET;
  byte local_61;
  int local_60;
  int local_5c;
  undefined local_58 [16];
  undefined local_48 [16];
  undefined4 local_38;
  undefined4 local_34;
  undefined4 local_30;
  undefined4 local_2c;
  undefined4 local_28;
  undefined4 local_24;
  undefined4 local_20;
  undefined4 local_1c;
  char local_18 [8];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_58 = (undefined  [16])0x0;
  local_48 = (undefined  [16])0x0;
  local_38 = 2;
  local_34 = 6;
  local_30 = 0xfffffffc;
  local_2c = 6;
  local_28 = 0;
  local_24 = 4;
  local_20 = 0xfffffffd;
  local_1c = 1;
  local_60 = 1;
  puts(
      "Interact with objects to advance your quest. Set all quest objective flags and turn in the qu est to get the real flag."
      );
  while (local_60 != 0) {
    printf("Current Values: [ ");
    for (local_5c = 0; local_5c < 8; local_5c = local_5c + 1) {
      printf("%d\t",(ulong)*(uint *)(local_58 + (long)local_5c * 4));
    }
    puts("]");
    print_menu();
    memset(local_18,0,8);
    fgets(local_18,8,stdin);
    __isoc99_sscanf(local_18,&DAT_00103250,&local_61);
    local_61 = local_61 - 0x30;
    switch(local_61) {
    case 0:
      local_60 = turn_in(local_58,&local_38);
      break;
    case 1:
      reset(local_58);
      break;
    case 2:
      rotate(local_58);
      break;
    case 3:
      increment(local_58);
      break;
    case 4:
      add_sub(local_58);
      break;
    case 5:
      modulo(local_58);
      break;
    case 6:
      swap(local_58);
      break;
    case 7:
      bitshift(local_58);
      break;
    case 8:
      flip_sign(local_58);
      break;
    case 0xbad1abe1:
      puts("That\'s not a valid action.\n");
      break;
    default:
      puts("That\'s not a valid action.\n");
    }
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

`handle_flag()`:
```shell
void handle_flag(long param_1)

{
  long in_FS_OFFSET;
  int local_258;
  int local_254;
  int local_248 [64];
  undefined4 local_148 [8];
  undefined local_128 [32];
  undefined local_108 [32];
  undefined local_e8 [32];
  undefined local_c8 [32];
  undefined local_a8 [32];
  undefined local_88 [32];
  undefined local_68 [32];
  undefined8 local_48;
  undefined8 local_40;
  undefined8 local_38;
  undefined7 local_30;
  undefined uStack_29;
  undefined7 uStack_28;
  undefined8 local_21;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_248[0] = 5;
  local_248[1] = 3;
  local_248[2] = 0;
  local_248[3] = 7;
  local_248[4] = 1;
  local_248[5] = 6;
  local_248[6] = 4;
  local_248[7] = 2;
  local_248[8] = 3;
  local_248[9] = 6;
  local_248[10] = 2;
  local_248[11] = 0;
  local_248[12] = 1;
  local_248[13] = 4;
  local_248[14] = 5;
  local_248[15] = 7;
  local_248[16] = 4;
  local_248[17] = 5;
  local_248[18] = 1;
  local_248[19] = 3;
  local_248[20] = 6;
  local_248[21] = 0;
  local_248[22] = 2;
  local_248[23] = 7;
  local_248[24] = 3;
  local_248[25] = 2;
  local_248[26] = 5;
  local_248[27] = 1;
  local_248[28] = 4;
  local_248[29] = 0;
  local_248[30] = 6;
  local_248[31] = 7;
  local_248[32] = 7;
  local_248[33] = 4;
  local_248[34] = 6;
  local_248[35] = 1;
  local_248[36] = 0;
  local_248[37] = 3;
  local_248[38] = 2;
  local_248[39] = 5;
  local_248[40] = 4;
  local_248[41] = 3;
  local_248[42] = 0;
  local_248[43] = 5;
  local_248[44] = 6;
  local_248[45] = 7;
  local_248[46] = 1;
  local_248[47] = 2;
  local_248[48] = 6;
  local_248[49] = 0;
  local_248[50] = 3;
  local_248[51] = 2;
  local_248[52] = 1;
  local_248[53] = 7;
  local_248[54] = 4;
  local_248[55] = 5;
  local_248[56] = 5;
  local_248[57] = 4;
  local_248[58] = 2;
  local_248[59] = 7;
  local_248[60] = 3;
  local_248[61] = 0;
  local_248[62] = 6;
  local_248[63] = 1;
  for (local_258 = 0; local_258 < 8; local_258 = local_258 + 1) {
    for (local_254 = 0; local_254 < 8; local_254 = local_254 + 1) {
      local_148[(long)local_254 + (long)local_258 * 8] =
           *(undefined4 *)(param_1 + (long)local_248[(long)local_254 + (long)local_258 * 8] * 4);
    }
  }
  local_48 = 0;
  local_40 = 0;
  local_38 = 0;
  local_30 = 0;
  uStack_29 = 0;
  uStack_28 = 0;
  local_21 = 0;
  transform(&local_48,local_248,local_148,&DAT_00103058,0x25,0,9);
  transform(&local_48,local_248 + 8,local_128,&DAT_00103058,0x20,9,5);
  transform(&local_48,local_248 + 0x10,local_108,&DAT_00103058,0x1a,0xe,6);
  transform(&local_48,local_248 + 0x18,local_e8,&DAT_00103058,0x15,0x14,5);
  transform(&local_48,local_248 + 0x20,local_c8,&DAT_00103058,0x10,0x19,5);
  transform(&local_48,local_248 + 0x28,local_a8,&DAT_00103058,0xd,0x1e,3);
  transform(&local_48,local_248 + 0x30,local_88,&DAT_00103058,6,0x21,7);
  transform(&local_48,local_248 + 0x38,local_68,&DAT_00103058,0,0x28,6);
  printf("Here\'s your reward: %s\n",&local_48);
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}
```

From `main()` the required quest state is clearly initialized:
```shell
local_38 = 2;  
local_34 = 6;  
local_30 = 0xfffffffc;  
local_2c = 6;  
local_28 = 0;  
local_24 = 4;  
local_20 = 0xfffffffd;  
local_1c = 1;
```

So the correct input to `handle_flag()` is: `[2, 6, -4, 6, 0, 4, -3, 1]`

I can use `gdb` to allocate the correct quest array and call the `handle_flag()` function to print the flag:
```shell
gdb ./brokenquest  
GNU gdb (Debian 16.3-1) 16.3  
Copyright (C) 2024 Free Software Foundation, Inc.  
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>  
This is free software: you are free to change and redistribute it.  
There is NO WARRANTY, to the extent permitted by law.  
Type "show copying" and "show warranty" for details.  
This GDB was configured as "x86_64-linux-gnu".  
Type "show configuration" for configuration details.  
For bug reporting instructions, please see:  
<https://www.gnu.org/software/gdb/bugs/>.  
Find the GDB manual and other documentation resources online at:  
<http://www.gnu.org/software/gdb/documentation/>.  
  
For help, type "help".  
Type "apropos word" to search for commands related to "word"...  
Reading symbols from ./brokenquest...  
(No debugging symbols found in ./brokenquest)  
(gdb) b main  
Breakpoint 1 at 0x1fed  
(gdb) run  
Starting program: /home/user/Desktop/ctf/texsaw2026/brokenquest/brokenquest  
[Thread debugging using libthread_db enabled]  
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".  
  
Breakpoint 1, 0x0000555555555fed in main ()  
(gdb) set $quest = (int[8]){2,6,-4,6,0,4,-3,1}  
(gdb) call (void)handle_flag((long)$quest)  
Here's your reward: texsaw{1t_ju5t_work5_m0r3_l1k3_!t_d0e5nt_w0rk}
```

`texsaw{1t_ju5t_work5_m0r3_l1k3_!t_d0e5nt_w0rk}`

___

## Switcheroo Read

**Author**: Written by Ben

**Category**: rev

**Description**: Whoopsie, some wild functions started switching my string. Please determine a string to fit their confusion. Flag format: `texsaw{string}` ex: `texsaw{confused_String}`

I get the challenge file:
```shell
file switcheroo  
switcheroo: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=8e4705b20693c97b80cd79e5f798dc865c486742, for GNU/Linux 3.2.0, stripped

./switcheroo  
Please make a compatible password: SOMETHING
```

I opened the binary in ghidra and found these functions:

`FUN_00401729()` (main validation logic):
```shell
void FUN_00401729(char *param_1)

{
  FUN_004012df(param_1,5);
  FUN_004012df(param_1,6);
  if (param_1[0xb] == 'o') {
    FUN_004012df(param_1,0xd);
    if (param_1[0xe] == 'R') {
      FUN_004012df(param_1,3);
      FUN_004012df(param_1,0x18);
      if ((*param_1 == -0x65) && ((byte)(param_1[0x1a] + 0x8dU) < 5)) {
        FUN_004012df(param_1,10);
        if ((param_1[8] == 'Y') && ((param_1[0xb] == 'Y' && ((byte)(param_1[0xc] + 0x8cU) < 4)))) {
          FUN_004012df(param_1,7);
          if ((param_1[0x14] == -0x4b) && (param_1[0xd] == 's')) {
            FUN_004013fd(param_1);
          }
        }
      }
    }
  }
  return;
```

`FUN_004012df()` (transformation function):
```shell
void FUN_004012df(long param_1,uint param_2)

{
  int iVar1;
  int local_10;
  int local_c;
  
  if ((param_2 & 1) == 0) {
    for (local_c = 0; local_c < (int)param_2; local_c = local_c + 1) {
      iVar1 = (int)(local_c * param_2) % 0x1b;
      *(char *)(param_1 + iVar1) = *(char *)(param_1 + iVar1) + (char)param_2;
    }
    FUN_004011b6(param_1,param_2);
  }
  else {
    FUN_004011b6(param_1,param_2);
    for (local_10 = 0; local_10 < (int)param_2; local_10 = local_10 + 1) {
      iVar1 = (int)(local_10 + param_2) % 0x1b;
      *(char *)(param_1 + iVar1) = *(char *)(param_1 + iVar1) - (char)param_2;
    }
  }
  return;
}
```

`FUN_004013fd()` (final validation):
```shell
void FUN_004013fd(char *param_1)

{
  int iVar1;
  long lVar2;
  char local_38;
  undefined local_37;
  undefined local_36;
  char local_35;
  undefined local_34;
  undefined local_33;
  char local_32;
  undefined local_31;
  undefined local_30;
  char local_2f;
  undefined local_2e;
  undefined local_2d;
  undefined local_2c;
  char local_2b;
  char local_2a;
  char local_29;
  char local_28;
  char local_27;
  char local_26;
  char local_25;
  char local_24;
  char local_23;
  char local_22;
  undefined local_21;
  int local_20;
  int local_1c;
  int local_18;
  int local_14;
  FILE *local_10;
  
  local_2b = *param_1 + -0x21;
  local_22 = (param_1[0x1a] + '\x06') * -2;
  local_2a = param_1[1] + -0x20;
  local_23 = param_1[8] + -7;
  local_29 = param_1[2] + -0x28;
  local_24 = param_1[9] + '\x14';
  local_28 = (param_1[3] + '\x04') * '\x02';
  local_25 = param_1[10] + '\b';
  local_27 = param_1[0xc] + '\x1c';
  local_26 = param_1[0xb] + -0x66;
  local_21 = 0;
  iVar1 = strcmp(&local_2b,s_README.txt_00404060);
  if (iVar1 != 0) {
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  local_10 = fopen(&local_2b,"rb");
  if (local_10 == (FILE *)0x0) {
    perror("fopen");
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  fread(&local_2c,1,1,local_10);
  local_2f = FUN_0040129c((int)(char)(-2 - param_1[5] / '\x02'));
  local_2e = FUN_0040129c((int)(char)(param_1[6] + '\x04'));
  local_2d = 0;
  lVar2 = strtol(&local_2f,(char **)0x0,0x10);
  local_14 = (int)lVar2;
  fread(&local_2c,1,1,local_10);
  local_32 = FUN_0040129c((int)(char)(param_1[7] + -0x2b));
  local_31 = FUN_0040129c((int)(char)(param_1[0x19] + -0x31));
  local_30 = 0;
  lVar2 = strtol(&local_32,(char **)0x0,0x10);
  local_18 = (int)lVar2;
  fread(&local_2c,1,1,local_10);
  local_35 = FUN_0040129c((int)(char)(param_1[0x18] + '\x05'));
  local_34 = FUN_0040129c((int)(char)('\b' - param_1[0x17] / '\x02'));
  local_33 = 0;
  lVar2 = strtol(&local_35,(char **)0x0,0x10);
  local_1c = (int)lVar2;
  fread(&local_2c,1,1,local_10);
  local_38 = FUN_0040129c((int)(char)(param_1[0x16] + -0xf));
  local_37 = FUN_0040129c((int)(char)(param_1[0x15] + -0x3d));
  local_36 = 0;
  lVar2 = strtol(&local_38,(char **)0x0,0x10);
  local_20 = (int)lVar2;
  if ((((local_1c == 0x61) && (local_18 == 0x34)) && (local_14 == 0x57)) && (local_20 == 0x29)) {
    printf("You have entered the flag");
  }
  return;
}
```

`FUN_004011b6()` (handles the string rotation operations used in `FUN_004012df`):
```shell
void FUN_004011b6(char *param_1,int param_2)

{
  char local_38 [40];
  undefined4 local_10;
  int local_c;
  
  local_10 = 0;
  strcpy(local_38,param_1);
  for (local_c = 0; local_c < 0x1b; local_c = local_c + 1) {
    param_1[(local_c + param_2) % 0x1b] = local_38[local_c];
  }
  param_1[0x1b] = '\0';
  return;
}
```

- Input 27 chars
- Apply transforms T5, T6, T13, T3, T24, T10, T7 (each T_k modifies string)
- After specific transforms, check certain indices equal specific chars/values
- Finally, extract 10 chars from final state to form "README.txt"
- Open that file, read 4 bytes, each must match hardcoded hex values

Solve script:
```shell
def rotate_right(s, k):
    k %= 27
    return s[-k:] + s[:-k]

def rotate_left(s, k):
    k %= 27
    return s[k:] + s[:k]

def apply_transform(s, k):
    """Apply the transformation T_k to the list s (values 0-255)."""
    s = s[:]  # copy
    if k % 2 == 0:  # even
        for i in range(k):
            idx = (i * k) % 27
            s[idx] = (s[idx] + k) & 0xFF
        s = rotate_right(s, k)
    else:  # odd
        s = rotate_right(s, k)
        for i in range(k):
            idx = (i + k) % 27
            s[idx] = (s[idx] - k) & 0xFF
    return s

def check_all(password_str):
    # Convert string to list of integers (0-255)
    s0 = [ord(c) for c in password_str]
    if len(s0) != 27:
        print("Password must be 27 characters.")
        return False

    # Apply transforms in order
    s1 = apply_transform(s0, 5)
    s2 = apply_transform(s1, 6)
    s3 = apply_transform(s2, 13)
    s4 = apply_transform(s3, 3)
    s5 = apply_transform(s4, 24)
    s6 = apply_transform(s5, 10)
    s7 = apply_transform(s6, 7)

    # Check all conditions
    # S2[11] == 'o'
    if s2[11] != ord('o'):
        print("Failed: S2[11] != 'o'")
        return False
    # S3[14] == 'R'
    if s3[14] != ord('R'):
        print("Failed: S3[14] != 'R'")
        return False
    # S5[0] == -101 (155 unsigned)
    if s5[0] != 155:  # -101 mod 256 = 155
        print("Failed: S5[0] != -101")
        return False
    # S5[26] in {115..119}
    if s5[26] not in range(115, 120):
        print("Failed: S5[26] not in {115..119}")
        return False
    # S6[8] == 'Y', S6[11] == 'Y'
    if s6[8] != ord('Y') or s6[11] != ord('Y'):
        print("Failed: S6[8] or S6[11] != 'Y'")
        return False
    # S6[12] in {116..119}
    if s6[12] not in range(116, 120):
        print("Failed: S6[12] not in {116..119}")
        return False
    # S7[13] == 's', S7[20] == -75 (181 unsigned)
    if s7[13] != ord('s') or s7[20] != 181:
        print("Failed: S7[13] != 's' or S7[20] != -75")
        return False

    # Construct the string for README.txt
    local_2b = (s7[0] - 33) & 0xFF
    local_2a = (s7[1] - 32) & 0xFF
    local_29 = (s7[2] - 40) & 0xFF
    local_28 = ((s7[3] + 4) * 2) & 0xFF
    local_27 = (s7[12] + 28) & 0xFF
    local_26 = (s7[11] - 102) & 0xFF
    local_25 = (s7[10] + 8) & 0xFF
    local_24 = (s7[9] + 20) & 0xFF
    local_23 = (s7[8] - 7) & 0xFF
    local_22 = ((s7[26] + 6) * -2) & 0xFF
    # Build the string in order
    built = [local_2b, local_2a, local_29, local_28, local_27,
             local_26, local_25, local_24, local_23, local_22]
    built_str = ''.join(chr(b) for b in built)
    if built_str != "README.txt":
        print(f"Failed: built string is '{built_str}', expected 'README.txt'")
        return False

    print("All checks passed!")
    return True

# Test the candidate
candidate = "texsaw{pAt1ence!!_W0rKn0w?}"
if check_all(candidate):
    print(f"Flag: {candidate}")
else:
    print("Candidate failed.")
```

Output:
```shell
./switcheroo  
Please make a compatible password: texsaw{pAt1ence!!_W0rKn0w?}  
You have entered the flag
```

`texsaw{pAt1ence!!_W0rKn0w?}`
