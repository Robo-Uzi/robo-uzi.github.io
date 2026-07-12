---
layout: post
title:  "Misc Challenges"
date:   2026-07-12 13:14:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /bronco-CTF-2026-misc/
---
* TOC
{:toc}

## Spot The Difference

**Category**: misc

**Author**: tiffany_ttn

**Description**: My friend said that she updated file1 to send me a top-secret message, but I don't get it. File2 is still just a bunch of random characters?

I get two files: 
```shell
file file2.txt  
file2.txt: ASCII text  

file file1.txt  
file1.txt: ASCII text

cat file2.txt | head  
x  
E  
C  
2  
V  
K  
w  
G  
b  
w  

cat file1.txt | head  
x  
e  
c  
2  
V  
K  
w  
G  
g  
w
```

The files are formatted with one character per line. I can align them index by index to see exactly what changed:
```python
python3  
Python 3.13.5 (main, Jun 13 2026, 14:18:01) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> with open("file1.txt", "r") as f1, open("file2.txt", "r") as f2:  
...     file1 = "".join(f1.read().split())  
...     file2 = "".join(f2.read().split())  
...  
... flag = ""  
... for c1, c2 in zip(file1, file2):  
...     if c1.lower() != c2.lower():  
...         flag += c2  
...  
... print(f"Flag: {flag}")  
...  
Flag: bronco{y@yyy_Y0u_f0und_m3!!}[
```

`bronco{y@yyy_Y0u_f0und_m3!!}`

___

## Zip, Zip, Hooray!

**Category**: misc

**Author**: .tidalw

**Description**: I was trying to compress my files and my script got a little carried away...

Can you help me find my original file?

Hint: the 7z files are password protected, and the password is the name of the first file inside

I get the challenge file `challenge.zip`:
```shell
file challenge.zip  
challenge.zip: gzip compressed data, was "layer1.tar", last modified: Sat Feb 28 02:39:52 2026, max compression, original size modulo 2^32 481280
```

It extracts to `layer1.tar`. Then to `layer2.bz2`. Then to `layer2`. `layer2` needs to be extracted with the password `layer4.zip`. This needs to be done repeatedly until the flag file is uncovered. I run this shell script:
```shell
#!/bin/bash
file="$1"

while [ -f "$file" ]; do
    type=$(file -b "$file")
    echo "Processing: $file ($type)"
    
    case "$type" in
        *gzip*)
            if tar -tzf "$file" &>/dev/null; then
                tar -xzf "$file"
                file=$(tar -tf "$file" | head -1)
            else
                newfile=$(file "$file" | grep -o 'was "[^"]*"' | cut -d'"' -f2)
                [ -z "$newfile" ] && newfile="${file%.gz}"
                gunzip -c "$file" > "$newfile"
                file="$newfile"
            fi
            ;;
        *tar*)
            tar -xf "$file"
            file=$(tar -tf "$file" | head -1)
            ;;
        *bzip2*)
            newfile="${file%.bz2}"
            bunzip2 -c "$file" > "$newfile"
            file="$newfile"
            ;;
        *7-zip*)
            # Get the first file's Path (second Path line)
            first=$(7z l -slt "$file" | grep '^Path =' | sed -n '2p' | cut -d'=' -f2- | sed 's/^ //')
            if [ -z "$first" ]; then
                echo "Error: no inner file found in 7z archive"
                break
            fi
            echo "> Password: $first"
            7z x -p"$first" "$file" >/dev/null || { echo "7z extraction failed"; break; }
            file="$first"
            ;;
        *Zip*)
            unzip -q "$file"
            # Get the first file from zip listing
            file=$(unzip -l "$file" | tail -n +4 | head -n -2 | head -1 | awk '{print $NF}')
            ;;
        *)
            echo "Unknown type: $type. Stopping"
            break
            ;;
    esac

    if [ ! -f "$file" ]; then
        echo "Expected '$file' not found. Stopping"
        break
    fi
done

echo "Final file: $file"
```

I run the script:
```shell
./extract.sh layer57.tar.gz  
Processing: layer57.tar.gz (gzip compressed data, was "layer57.tar", last modified: Sat Feb 28 02:39:43 2026, max compression, original size modulo 2^32 430080)  
Processing: layer58.bz2 (bzip2 compressed data, block size = 900k)  
Processing: layer58 (7-zip archive data, version 0.4)  
> Password: layer60.zip

... etc ...

> Password: layer996.zip  
Processing: layer996.zip (Zip archive data, made by v2.0, extract using at least v2.0, last modified Feb 27 2026 18:37:36, uncompressed size 608, method=deflate)
Processing: layer997.tar.gz (gzip compressed data, was "layer997.tar", last modified: Sat Feb 28 02:37:36 2026, max compression, original size modulo 2^32 10240)  
Processing: layer998.bz2 (bzip2 compressed data, block size = 900k)  
Processing: layer998 (7-zip archive data, version 0.4)  
> Password: layer1000.zip  
Processing: layer1000.zip (Zip archive data, made by v2.0, extract using at least v2.0, last modified Feb 27 2026 18:37:36, uncompressed size 31, method=deflate)  
Processing: flag.txt (ASCII text, with no line terminators)  
Unknown type: ASCII text, with no line terminators — stopping  
Final file: flag.txt
```

```shell
cat flag.txt  
bronco{i_h4te_f1l3_c0mpr3ssi0n}
```

`bronco{i_h4te_f1l3_c0mpr3ssi0n}`

___

## Sliced and Diced

**Category**: misc

**Author**: yoshie878

**Description**: My questionable QR code got thrown through a rift. Unslice and undice to find the flag on the other side!

I get the challenge image `sliced-and-diced.png`:

![Alt text](/images/sliced-and-diced.png)

I used gimp to put the qr code back together:

![Alt text](/images/unsliced-and-diced.png)

The qr code scanned and output this canva link: [https://www.canva.com/design/DAHCqL0Sd-E/WowFDvU8s09qQU8FkVpntw/view?utm_content=DAHCqL0Sd-E&utm_campaign=designshare&utm_medium=link2&utm_source=uniquelinks&utlId=h8ae379a391](https://www.canva.com/design/DAHCqL0Sd-E/WowFDvU8s09qQU8FkVpntw/view?utm_content=DAHCqL0Sd-E&utm_campaign=designshare&utm_medium=link2&utm_source=uniquelinks&utlId=h8ae379a391)

On canva it looked like the same image however I could do `ctrl+a` and select the entire flag string:

![Alt text](/images/broncoweb9.png)

Then I can just copy and paste it! 

`bronco{th3_h1dd3n_cu3}`
