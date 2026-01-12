---
layout: post
title:  "court_jester"
date:   2026-01-12 01:23:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /scarletCTF-court_jester/
---

**Title**: court_jester

**Category**: Rev

**Author**: s0s.sh

**Description**: I reside as the Queen of this data kingdom. Prithee, I implore you to remedy the ailments of our poor beloved court jester. In the days since he imbibed an entire cask of old mead I suspect was fermented from a bad batch of barley microkernels, he has not been himself and, well, juggles data all wrong! Help us I plead, our kingdom depends on YOU, traveller!

I get the challenge file:
```shell
file court_jester  
court_jester: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), statically linked, no section header
```
It is statically linked with no section header. It makes static analysis difficult but data will eventually be written to stdout.

Running the binary shows some ASCII art. Running `strace` shows this:
```shell
strace -f ./court_jester  
execve("./court_jester", ["./court_jester"], 0x7ffccd72fae8 /* 66 vars */) = 0  
open("/proc/self/exe", O_RDONLY)        = 3  
mmap(NULL, 3093, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f0d1a565000  
mprotect(0x7f0d1a565000, 3093, PROT_READ|PROT_EXEC) = 0  
readlink("/proc/self/exe", "/home/user/Desktop/ctf/scarletCTF"..., 4095) = 45  
mmap(0x7f0d1a56c000, 16432, PROT_NONE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f0d1a56c000  
mmap(0x7f0d1a56c000, 2328, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f0d1a56c000  
mprotect(0x7f0d1a56c000, 2328, PROT_READ) = 0  
mmap(0x7f0d1a56d000, 1337, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0x1000) = 0x7f0d1a56d000  
mprotect(0x7f0d1a56d000, 1337, PROT_READ|PROT_EXEC) = 0  
mmap(0x7f0d1a56e000, 1168, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0x2000) = 0x7f0d1a56e000  
mprotect(0x7f0d1a56e000, 1168, PROT_READ) = 0  
mmap(0x7f0d1a56f000, 4128, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0x2000) = 0x7f0d1a56f000  
mprotect(0x7f0d1a56f000, 4128, PROT_READ|PROT_WRITE) = 0  
open("/lib64/ld-linux-x86-64.so.2", O_RDONLY) = 4  
read(4, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0 \253\1\0\0\0\0\0"..., 1024) = 1024  
mmap(NULL, 217088, PROT_NONE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f0d1a530000  
mmap(0x7f0d1a530000, 3416, PROT_READ, MAP_PRIVATE|MAP_FIXED, 4, 0) = 0x7f0d1a530000  
mmap(0x7f0d1a531000, 151601, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED, 4, 0x1000) = 0x7f0d1a531000  
mmap(0x7f0d1a557000, 39956, PROT_READ, MAP_PRIVATE|MAP_FIXED, 4, 0x27000) = 0x7f0d1a557000  
mmap(0x7f0d1a561000, 12560, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED, 4, 0x31000) = 0x7f0d1a561000  
close(4)                                = 0  
brk(0x7f0d1a571000)                     = 0x555556dc5000  
munmap(0x7f0d1a571000, 6037)            = 0  
mmap(NULL, 4096, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f0d1a572000  
close(3)                                = 0  
munmap(0x7f0d1a565000, 3093)            = 0  
brk(NULL)                               = 0x555556dc5000  
  
  
  
             (0x2c) -  
                       '  
                        '  
               @_  _    '  
                )\/(@    '  
              __(/ \--._  
             (,-.---'--'@  
              @ )0_0(     _  
                ('-')    (_)  
           '    _\Y/_  
           ' .-'-\-/-'-._  '  
           _ /    '*      '  
          (_)  /)  *    .-.))>'  
            ._/  \__*_ /\__'.  
        '<((_'    |__H/  \__\  
                  /   ,_/ |_|  
                  )-- /   |x|  
                  \ _/    (_ x  
                  /_/       \_\@  
                 /_/  
                /_/  
               /x/  
              (_ x  
                \_\@  
  
  
  
  
  
) = 758  
write(1, "\n", 1  
)                       = 1  
pipe2([3, 4], 0)                        = 0  
pipe2([5, 6], 0)                        = 0  
rt_sigaction(SIGCHLD, {sa_handler=SIG_IGN, sa_mask=[CHLD], sa_flags=SA_RESTORER|SA_RESTART, sa_restorer=0x7f0d1a36e050}, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0  
clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLDstrace: Process 89711 attached  
, child_tidptr=0x7f0d1a32fa10) = 89711  
[pid 89710] close(5)                    = 0  
[pid 89710] close(4)                    = 0  
[pid 89710] write(6, "\264\310\265\330\245\346\217\302\225\350\226\355\211\356\325\302\237\362\223\302", 20) = 20  
[pid 89710] read(3,  <unfinished ...>  
[pid 89711] set_robust_list(0x7f0d1a32fa20, 24) = 0  
[pid 89711] close(3)                    = 0  
[pid 89711] close(6)                    = 0  
[pid 89711] read(5, "\264\310\265\330\245\346\217\302\225\350\226\355\211\356\325\302\237\362\223\302", 20) = 20  
[pid 89711] write(4, "~y\177ioWEs_Y\\\\C_\37sUCYs", 20 <unfinished ...>  
[pid 89710] <... read resumed>"~y\177ioWEs_Y\\\\C_\37sUCYs", 20) = 20  
[pid 89711] <... write resumed>)        = 20  
[pid 89710] write(6, "\312\220\216\217\217\202\235\211\311\201\245\210\237\272\266\256\264\242\274\260", 20) = 20  
[pid 89710] read(3,  <unfinished ...>  
[pid 89711] read(5, "\312\220\216\217\217\202\235\211\311\201\245\210\237\272\266\256\264\242\274\260", 20) = 20  
[pid 89711] write(4, "\34YXFYKK@\37HsAIs`gbkjy", 20 <unfinished ...>  
[pid 89710] <... read resumed>"\34YXFYKK@\37HsAIs`gbkjy", 20) = 20  
[pid 89711] <... write resumed>)        = 20  
[pid 89710] write(6, "\265\23\277s\337}\301\177\325\34\311e\312n\323\37\302f\315V", 20) = 20  
[pid 89710] read(3,  <unfinished ...>  
[pid 89711] read(5, "\265\23\277s\337}\301\177\325\34\311e\312n\323\37\302f\315V", 20) = 20  
[pid 89711] write(4, "\37\24\25tuzkx\177\33cb`iy\30hagQ", 20 <unfinished ...>  
[pid 89710] <... read resumed>"\37\24\25tuzkx\177\33cb`iy\30hagQ", 20) = 20  
[pid 89711] <... write resumed>)        = 20  
[pid 89710] write(1, ".,, ., ,, ,  ,.,, (0x2c) ,,.,   "..., 46.,, ., ,, ,  ,.,, (0x2c) ,,.,   .,  ,. . .,,  
  
) = 46  
[pid 89710] close(6)                    = 0  
[pid 89710] exit_group(0)               = ?  
[pid 89711] close(4 <unfinished ...>  
[pid 89710] +++ exited with 0 +++  
  
  
  
  
  
  
             (0x2c) -  
                       '  
                        '  
               @_  _    '  
                )\/(@    '  
              __(/ \--._  
             (,-.---'--'@  
              @ )0_0(     _  
                ('-')    (_)  
           '    _\Y/_  
           ' .-'-\-/-'-._  '  
           _ /    '*      '  
          (_)  /)  *    .-.))>'  
            ._/  \__*_ /\__'.  
        '<((_'    |__H/  \__\  
                  /   ,_/ |_|  
                  )-- /   |x|  
                  \ _/    (_ x  
                  /_/       \_\@  
                 /_/  
                /_/  
               /x/  
              (_ x  
                \_\@  
  
  
  
  
  
  
.,, ., ,, ,  ,.,, (0x2c) ,,.,   .,  ,. . .,,  
  
) = 805  
[pid 89925] +++ exited with 0 +++  
+++ exited with 0 +++
```
The flag is generated at runtime. A child process transforms bytes and the parent prints the transformed output 3 times. Indicated by the `write(4, "...", 20)` strings. 

Run `strace` and put it in a file:
```shell
strace -f -e write ./court_jester 2>&1 | tee trace.txt
```

Get the desired output:
```shell
grep 'write(4' trace.txt | sed 's/.*"\(.*\)",.*/\1/' > decoded.txt

cat decoded.txt  
~y\177ioWEs_Y\\\\C_\37sUCYs  
\34YXFYKK@\37HsAIs`gbkjy  
\37\24\25tuzkx\177\33cb`iy\30hagQ
```

Convert it to raw bytes:
```python
python3 - << 'EOF'
import codecs

out = b""
for line in open("decoded.txt", "r"):
    s = codecs.decode(line.strip(), 'unicode_escape')
    out += s.encode('latin1')

open("decoded.bin", "wb").write(out)
print("Total bytes:", len(out))
print("Raw bytes:", out)
EOF
```

XOR to get the flag:
```python
#!/usr/bin/env python3

data = open("decoded.bin", "rb").read()

def is_printable(bs):
    return all(32 <= b <= 126 for b in bs)

def try_candidate(bs, key):
    try:
        s = bs.decode()
        if "RUSEC{" in s:
            print(f"Flag: {s}")
            print(f"XOR key: 0x{key:02x}")
            exit(0)
    except:
        pass

for k in range(256):
    out = bytes(b ^ k for b in data)
    if is_printable(out):
        try_candidate(out, k)
```

Output:
```shell
python3 solve.py  
Flag: RUSEC{i_suppos3_you_0utjuggl3d_me_LKNGFU389XYVGTS7ONLEU4DMK}  
XOR key: 0x2c
```
I realized I didnt need to brute force the XOR key. The ASCII art jester is juggling the byte. Either way. Got the flag!

`RUSEC{i_suppos3_you_0utjuggl3d_me_LKNGFU389XYVGTS7ONLEU4DMK}`