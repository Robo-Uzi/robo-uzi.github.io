---
layout: post
title: "Reverse Engineering Challenges"
date:   2026-09-07 15:14:00 -0400
author: uzi
tags: [CTF]
permalink: /nns-ctf-2026-rev/
---
* TOC
{:toc}

## No Strings Attached

**Author**: hoover

**Category**: Reverse Engineering

**Description**: New to reverse engineering? This beginner challenge is an introduction to tracing the library calls a Linux program makes.

The provided `x86-64` ELF asks you to guess a passphrase. The passphrase is the flag, but it is not written down anywhere in the file. Running `strings` on the binary only gets you `guess:`, `correct` and `rejected`. The program builds the real passphrase in memory first, and only then hands it to a function in the C standard library to compare against yours.

Run the program under a library call tracer such as `ltrace` and watch the comparison. Your goal is to read the passphrase out of the arguments the program passes to it.

I get the challenge file:
```shell
file no-strings-attached  
no-strings-attached: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=7d8306bb906fec01cafc9e3079d6ffbdb6e00e35, for GNU/Linux 4.4.0, not stripped
```

Running the program with `ltrace`:
```shell
ltrace -s 100 ./no-strings-attached -v  
write(1, "guess: ", 7guess: )                                            = 7  
read(0guess  
, "guess\n", 127)                                                        = 6  
strcmp("guess", "NNS{n0_str1ngs_1n_7h3_b1n4ry_bu7_ltr4c3_s4w_7h3_c0mp4r3}") = 25  
write(1, "rejected\n", 9rejected  
)                                                                        = 9  
+++ exited (status 2) +++

./no-strings-attached  
guess: NNS{n0_str1ngs_1n_7h3_b1n4ry_bu7_ltr4c3_s4w_7h3_c0mp4r3}  
correct
```

`NNS{n0_str1ngs_1n_7h3_b1n4ry_bu7_ltr4c3_s4w_7h3_c0mp4r3}`

___

## Open Secret

**Author**: hoover

**Category**: Reverse Engineering

**Description**: New to reverse engineering? This beginner challenge is an introduction to tracing the system calls a Linux program makes.

The provided `x86-64` ELF wants a license file before it will do anything, but it will not tell you which one. The path is not written down in the binary either and running `strings` on it only gets you `no license`. The program builds the path in memory first, and only then asks the kernel to open it.

This one talks to the kernel directly, so a library call tracer has nothing to show you. Run the program under a system call tracer such as `strace` instead, watch the call that opens the file, and read the path out of its arguments. Your goal is to create the file it is looking for and run the program again.

I get the challenge file:
```shell
file open-secret  
open-secret: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, BuildID[sha1]=00736964b5cf872c1b75f1b925843fb968d433fc, not stripped
```

Running `strace` once:
```shell
strace ./open-secret  
execve("./open-secret", ["./open-secret"], 0x7ffc19a9b950 /* 69 vars */) = 0  
openat(AT_FDCWD, "/home/user/.config/nns/key", O_RDONLY) = -1 ENOENT (No such file or directory)  
write(1, "no license\n", 11no license  
)            = 11  
exit(1)                                 = ?  
+++ exited with 1 +++
```

It tries to open the file `/home/user/.config/nns/key`. I created that file and put something random inside:
```shell
cat /home/user/.config/nns/key  
anything
```

Running `strace` again:
```shell
strace ./open-secret  
execve("./open-secret", ["./open-secret"], 0x7ffc3ab803c0 /* 69 vars */) = 0  
openat(AT_FDCWD, "/home/user/.config/nns/key", O_RDONLY) = 3  
close(3)                                = 0  
write(1, "NNS{7h3_p47h_w4s_h1dd3n_bu7_s7r4"..., 49NNS{7h3_p47h_w4s_h1dd3n_bu7_s7r4c3_s4w_7h3_0p3n}  
) = 49  
exit(0)                                 = ?  
+++ exited with 0 +++
```

`NNS{7h3_p47h_w4s_h1dd3n_bu7_s7r4c3_s4w_7h3_0p3n}`