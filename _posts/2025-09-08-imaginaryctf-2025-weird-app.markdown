---
layout: post
title:  "weird-app"
date:   2025-09-08 21:34:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /imaginaryctf-2025-weird-app/
---

**Title:** weird-app

**Category:** Reversing

**Author:** cleverbear57

**Description:** I made this weird android app, but all it gave me was this .apk file. Can you get the flag from it?

Attached is the file `weird.zip`. I unzip it and get the apk:
```shell
file app-debug.apk  
app-debug.apk: Zip archive data, at least v0.0 to extract, compression method=deflate
```

I run the apk on [https://www.myandroid.org/android-manager-to-run-apk-online/](https://www.myandroid.org/android-manager-to-run-apk-online/). When it loads, all I can see is this text: `Transformed flag: idvi+1{s6e3{)arg2zv[moqa905+`.

I decompile the apk to find how to flag was manipulated:
```shell
apktool d app-debug.apk  
I: Using Apktool 2.12.0 on app-debug.apk with 2 threads  
I: Baksmaling classes.dex...  
I: Loading resource table...  
I: Decoding file-resources...  
I: Loading resource table from file: /home/user/.local/share/apktool/framework/1.apk  
I: Decoding values */* XMLs...  
I: Decoding AndroidManifest.xml with resources...  
I: Baksmaling classes3.dex...  
I: Baksmaling classes2.dex...  
I: Baksmaling classes4.dex...  
I: Baksmaling classes5.dex...  
I: Copying original files...  
I: Copying lib...  
I: Copying unknown files...
```

I find one file that contains the transformed flag:
```shell
grep -R "idvi+1{" smali*  
smali_classes4/com/example/test2/MainActivityKt.smali:    const-string v4, "Transformed flag: idvi+1{s6e3{)arg2zv[moqa905+"
```

I run `cat smali_classes4/com/example/test2/MainActivityKt.smali` and read the code.

It defines three groups of characters:
```shell
alpha = "abcdefghijklmnopqrstuvwxyz"
nums = "0123456789"
spec = "!@#$%^&*()_+{}[]|"
```

A function loops over the flag and checks each character (and which group it belongs to).

- Transforming letters: 

1: Find the characters index in `alpha`.

2: Add the position in the flag string.

3: Modulo 26 (length of alphabet).

4: Append the resulting character to the output.

- Transforming numbers: 

1: Find the numbers index in `nums`.

2: Multiply the index of the character in the string by 2 and add the position in `nums`.

3: Modulo 10 (length of nums).

4: Append the resulting character.

- Transforming speacial characters: 

1: Find the characters index in `spec`.

2: Add the square of the character index in the flag.

3: Modulo 15 (length of spec).

4: Append the resulting character.

```python
transformed = "idvi+1{s6e3{)arg2zv[moqa905+"

alpha = "abcdefghijklmnopqrstuvwxyz"
nums  = "0123456789"
spec  = "!@#$%^&*()_+{}[]|"

flag = []

for i, c in enumerate(transformed):
    if c in alpha:
        ind = (alpha.index(c) - i) % len(alpha)
        flag.append(alpha[ind])
    elif c in nums:
        ind = (nums.index(c) - 2*i) % len(nums)
        flag.append(nums[ind])
    elif c in spec:
        ind = (spec.index(c) - i*i) % len(spec)
        flag.append(spec[ind])
    else:
        flag.append(c)

print(''.join(flag))
```
`ictf{1_l0v3_@ndr0id_stud103}`