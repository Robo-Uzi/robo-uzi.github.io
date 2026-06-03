---
layout: post
title:  "Misc Challenges"
date:   2026-06-03 05:46:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /BYU-CTF-2026-misc/
---
* TOC
{:toc}

## Easy

**Category**: Misc

**Author**: overllama

**Description**: Okay, easy mode is turned on. You know how to use bash, right? `chals.cyberjousting.com:1370`

I connect with `nc`:
```shell
nc chals.cyberjousting.com 1370  
Run anything you want! With... some modifications, anyways  
$ help  
Input:  
help  
Output:  
GNU bash, version 5.2.37(1)-release (x86_64-pc-linux-gnu)  
These shell commands are defined internally.  Type `help' to see this list.  
Type `help name' to find out more about the function `name'.  
Use `info bash' to find out more about the shell in general.  
Use `man -k' or `info' to find out more about commands not in this list.  
  
A star (*) next to a name means that the command is disabled.  
  
job_spec [&]                            history [-c] [-d offset] [n] or hist>  
(( expression ))                        if COMMANDS; then COMMANDS; [ elif C>  
. filename [arguments]                  jobs [-lnprs] [jobspec ...] or jobs >  
:                                       kill [-s sigspec | -n signum | -sigs>  
[ arg... ]                              let arg [arg ...]  
[[ expression ]]                        local [option] name[=value] ...  
alias [-p] [name[=value] ... ]          logout [n]  
bg [job_spec ...]                       mapfile [-d delim] [-n count] [-O or>  
bind [-lpsvPSVX] [-m keymap] [-f file>  popd [-n] [+N | -N]  
break [n]                               printf [-v var] format [arguments]  
builtin [shell-builtin [arg ...]]       pushd [-n] [+N | -N | dir]  
caller [expr]                           pwd [-LP]  
case WORD in [PATTERN [| PATTERN]...)>  read [-ers] [-a array] [-d delim] [->  
cd [-L|[-P [-e]] [-@]] [dir]            readarray [-d delim] [-n count] [-O >  
command [-pVv] command [arg ...]        readonly [-aAf] [name[=value] ...] o>  
compgen [-abcdefgjksuv] [-o option] [>  return [n]  
complete [-abcdefgjksuv] [-pr] [-DEI]>  select NAME [in WORDS ... ;] do COMM>  
compopt [-o|+o option] [-DEI] [name .>  set [-abefhkmnptuvxBCEHPT] [-o optio>  
continue [n]                            shift [n]  
coproc [NAME] command [redirections]    shopt [-pqsu] [-o] [optname ...]  
declare [-aAfFgiIlnrtux] [name[=value>  source filename [arguments]  
dirs [-clpv] [+N] [-N]                  suspend [-f]  
disown [-h] [-ar] [jobspec ... | pid >  test [expr]  
echo [-neE] [arg ...]                   time [-p] pipeline  
enable [-a] [-dnps] [-f filename] [na>  times  
eval [arg ...]                          trap [-lp] [[arg] signal_spec ...]  
exec [-cl] [-a name] [command [argume>  true  
exit [n]                                type [-afptP] name [name ...]  
export [-fn] [name[=value] ...] or ex>  typeset [-aAfFgiIlnrtux] name[=value>  
false                                   ulimit [-SHabcdefiklmnpqrstuvxPRT] [>  
fc [-e ename] [-lnr] [first] [last] o>  umask [-p] [-S] [mode]  
fg [job_spec]                           unalias [-a] name [name ...]  
for NAME [in WORDS ... ] ; do COMMAND>  unset [-f] [-v] [-n] [name ...]  
for (( exp1; exp2; exp3 )); do COMMAN>  until COMMANDS; do COMMANDS-2; done  
function name { COMMANDS ; } or name >  variables - Names and meanings of so>  
getopts optstring name [arg ...]        wait [-fn] [-p var] [id ...]  
hash [-lr] [-p pathname] [-dt] [name >  while COMMANDS; do COMMANDS-2; done  
help [-dms] [pattern ...]               { COMMANDS ; }  
$
```
You don't have things like `ls`, `cat`, or `env`. 

Eventually I figured out that `printf${IFS}'%s\n'${IFS}*` will list files (spaces were being stripped, and `${IFS}` works as a space):
```shell
nc chals.cyberjousting.com 1370  
Run anything you want! With... some modifications, anyways  
$ printf${IFS}'%s\n'${IFS}*  
Input:  
printf${IFS}'%s\n'${IFS}*  
Output:  
bashnflag.txtnrunn  
$
```

I read the flag with redirection: `x=$(<flag.txt);echo${IFS}$x`:
```shell
nc chals.cyberjousting.com 1370  
Run anything you want! With... some modifications, anyways  
$ x=$(<flag.txt);echo${IFS}$x  
Input:  
x=$(<flag.txt);echo${IFS}$x  
Output:  
byuctf{g0_t0_j41l_a60941}  
$
```

`byuctf{g0_t0_j41l_a60941}`

___

## Inception

**Category**: Misc

**Author**: The Camel

**Description**: I found this weird file on my computer. I tried opening it, but there were some problems.

I download the challenge file. `file` says its a png:
```shell
file inception  
inception: PNG image data, 273 x 160, 8-bit/color RGB, non-interlaced
```

I put the file into [cyberchef](https://gchq.github.io/CyberChef/#recipe=Render_Image('Raw')&input=iVBORw0KGgoAAAANSUhEUgAAAREAAACgCAIAAACkF8%2B9AAAC/klEQVR42u3cwW3CMBSAYTZAnaGXHjpMz5x77iQM0XEYopdO0kZCygEqSuz3bId80ndAKoHi%2BIc4Qez2T8%2BTl9e38w3gT3MjO83AsmamW8D9fM5AxbHZ4XScGSM27iIHzYBmQDMwejOA6zOgGdAMaAY0A5rRDGgGNAOaAc0EeP/4vBB1ZzJ208M/qWbQjGY0o5ltNnNjOBbFYzY/3vQd7Z1RM2hGM2hm482Eb369D8rWTuH7svIpFm11/adFW3WZx5rRjGY0oxnNaKb9oWrZuITP2i6rqbznqizEuWbNaEYzmtGMZqxnuqxnyg7lx1/PlL0X%2BB6AZjSjGc1oRjOa6bsSGG01lffa8wY8b2ZrRjOa0YxmNKOZwZtpeUW/wSXwqHncYD3T8guUfb98oBnNaEYzmtGM352Bvd%2BdAc2AZjQDmgHNgGZAM6AZQDMwbDPTY0IvPmfAsRloBjQDaAY0A5oBzYBmQDPGBTQDmgHNgGZAM6CZ9Ga%2Bvn/OHnUQK1/gvHnZ44QPb5cHXNEk0YxmNKMZzWhGMw/ZzI2t%2Bk6yqEde9F6gGc1oRjOa0YxmOjZzz2iWDXTl7rn%2Bx6LunLcKCr9P2f4q2ymVY6gZzWhGM5rRjGaSjmujZn/eVlGzv2VplaMRtb%2BsZzSjGc1oRjOaWd255rJD3jU2s8b1TFlpZYtDzWhGM5rRjGY0M3gzeac1w6sOv9jf8jxy3v7KG17NaEYzmtGMZjQTcq657Lpy5VYNLkt3%2BVZi3rX5ls2UnemunFqa0YxmNKMZzWyzGdj73RnQjGZAM7CiZg6nI4zG5ww4NgPNgGZAM5oBzYBmQDOgGdAMoBnQDGgGNAOaAc1oBjQDmgHNgGZAM4BmQDOgGdAMaAY0A2gGNAOaAc2AZkAzmgHNgGZAM6AZ0AygGdAMaAY0A5oBzWgGNAOaAc2AZkAzgGZAM6AZ0AxoBjQDaAY0A5oBzYBmQDOagYpmDqfjzBixcRc5aAY0A5qB0ZsB/j8HMN0C7udzBlyfgcxmfgEVgduJwTn/4AAAAABJRU5ErkJgglBLAwQUAAAACAAEu7Rcz5sMGBsAAABWAAAACAAAAGRhdGEuYmluS8tJTFcoSCwqUTDisiUAuLgUFEoyjLkIKwQAUEsBAhQDFAAAAAgABLu0XM%2BbDBgbAAAAVgAAAAgAAAAAAAAAAAAAAIABAAAAAGRhdGEuYmluUEsFBgAAAAABAAEANgAAAEEAAAAAACVQREYtMS40CiXi48/TCjEgMCBvYmoKPDwgL1R5cGUgL0NhdGFsb2cgL1BhZ2VzIDIgMCBSID4%2BCmVuZG9iagoyIDAgb2JqCjw8IC9UeXBlIC9QYWdlcyAvS2lkcyBbMyAwIFJdIC9Db3VudCAxID4%2BCmVuZG9iagozIDAgb2JqCjw8IC9UeXBlIC9QYWdlIC9QYXJlbnQgMiAwIFIgL01lZGlhQm94IFswIDAgNjEyIDc5Ml0KICAgL0NvbnRlbnRzIDQgMCBSIC9SZXNvdXJjZXMgPDwgL0ZvbnQgPDwgL0YxIDUgMCBSID4%2BID4%2BID4%2BCmVuZG9iago0IDAgb2JqCjw8IC9MZW5ndGggODQgL0ZpbHRlciAvRmxhdGVEZWNvZGUgPj4Kc3RyZWFtCnjacwrh0nczVDA0VAhJ4zI1UDA3MVAISeHSSMtJTFcoSCwqUTDWVAjJ4jJQ0DUyAsvYEgAw5cZgg8CGm4AM14hPKzJOzK7FIg%2B2nDiDXUMAZ3kpRQplbmRzdHJlYW0KZW5kb2JqCjUgMCBvYmoKPDwgL1R5cGUgL0ZvbnQgL1N1YnR5cGUgL1R5cGUxIC9CYXNlRm9udCAvSGVsdmV0aWNhID4%2BCmVuZG9iagp4cmVmCjAgNgowMDAwMDAwMDAwIDY1NTM1IGYgCjAwMDAwMDA5NzkgMDAwMDAgbiAKMDAwMDAwMTAyOCAwMDAwMCBuIAowMDAwMDAxMDg1IDAwMDAwIG4gCjAwMDAwMDEyMTQgMDAwMDAgbiAKMDAwMDAwMTM2OSAwMDAwMCBuIAp0cmFpbGVyCjw8IC9TaXplIDYgL1Jvb3QgMSAwIFIgPj4Kc3RhcnR4cmVmCjE0MzkKJSVFT0YK) and render an image to get the first part of the flag:

![Alt text](/images/theflagrendered.png)

Run `binwalk` to get a zip archive with part 2:
```shell
binwalk -e inception  
  
DECIMAL       HEXADECIMAL     DESCRIPTION  
--------------------------------------------------------------------------------  
41            0x29            Zlib compressed data, best compression  
823           0x337           Zip archive data, at least v2.0 to extract, compressed size: 27, uncompressed size: 86, name: data.bin  
1267          0x4F3           Zlib compressed data, best compression  
  
WARNING: One or more files failed to extract: either no utility was found or it's unimplemented  
  

cd _inception.extracted  

ls -la  
total 152  
drwxrwxr-x 1 user user     70 May 30 02:55 .  
drwxrwxr-x 1 user user    278 May 30 02:55 ..  
-rw-rw-r-- 1 user user 131200 May 30 02:55 29  
-rw-rw-r-- 1 user user   1582 May 30 02:55 29.zlib  
-rw-rw-r-- 1 user user    800 May 30 02:55 337.zip  
-rw-rw-r-- 1 user user    178 May 30 02:55 4F3  
-rw-rw-r-- 1 user user    356 May 30 02:55 4F3.zlib  
-rw------- 1 user user     86 May 20 23:24 data.bin  

cat data.bin  
flag part 2  
================================  
  
th3  
  
================================
```

The file `4F3` contained part 3 of the flag:
```shell
file *  
29:       data  
29.zlib:  zlib compressed data  
337.zip:  Zip archive data, at least v2.0 to extract, compression method=deflate
4F3:      ASCII text  
4F3.zlib: zlib compressed data  
data.bin: ASCII text

cat 4F3  
BT  
/F1 11 Tf  
50 740 Td  
(flag part 3) Tj  
0 -22 Td  
(================================) Tj  
0 -30 Td  
/F1 14 Tf  
(_fr3ak}) Tj  
0 -30 Td  
/F1 11 Tf  
(================================) Tj  
ET
```

`byuctf{wh4t_th3_fr3ak}`

___

## Chromatic

**Category**: Misc

**Author**: The Camel

**Description**: Red.

I get the challenge file:
```shell
file chromatic.mp4  
chromatic.mp4: ISO Media, MP4 Base Media v1 [ISO 14496-12:2003]
```

The video is 54 seconds long at 30 FPS. Every 30 frames (every 1 second) the entire screen is a solid color. Only the red channel changes. Each red channel value is an ASCII code for one character of the flag. Reading one value per 30 frame block and decoding as ASCII reveals the flag:
```python
#!/usr/bin/env python3
import sys
import cv2

def extract_flag(video_path: str) -> str:
    cap = cv2.VideoCapture(video_path)
    if not cap.isOpened():
        raise RuntimeError(f"Cannot open video: {video_path}")

    fps = int(round(cap.get(cv2.CAP_PROP_FPS)))
    print(f"{int(cap.get(cv2.CAP_PROP_FRAME_COUNT))} frames @ {fps} FPS")

    red_values = []
    while True:
        ret, frame = cap.read()
        if not ret:
            break
        red_values.append(int(frame[0, 0, 2]))  # red channel
    cap.release()

    chars = []
    for i in range(0, len(red_values), fps):
        chunk = red_values[i:i + fps]
        chars.append(chr(chunk[0]))  # convert ASCII code to character

    flag = "".join(chars)
    return flag

if __name__ == "__main__":
    path = sys.argv[1] if len(sys.argv) > 1 else "chromatic.mp4"
    flag = extract_flag(path)
    print(f"Flag: {flag}")
```

Output:
```shell
python3 solve2.py  
1620 frames @ 30 FPS  
Flag: byuctf{It's_all_red_I_really_thought_it_would_be_more}
```

`byuctf{It's_all_red_I_really_thought_it_would_be_more}`
