---
layout: post
title:  "Crypto Challenges"
date:   2026-05-28 13:58:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /SecLeaf-Q2-CTF-2026-crypto/
---
* TOC
{:toc}

## military_grade_encryption

**Category**: Cryptography

**Description**: We intercepted an encrypted military transmission during routine monitoring. Analysts were unable to identify the encryption scheme used. Can you recover the hidden message? Flag format: `SecLeaf{}`

```shell
cat encrypted.txt  
U2VjTGVhZntiNDUzNjRfMXNfbjB0XzNuY3J5cHQxMG59  

echo "U2VjTGVhZntiNDUzNjRfMXNfbjB0XzNuY3J5cHQxMG59" | base64 -d  
SecLeaf{b45364_1s_n0t_3ncrypt10n}
```

`SecLeaf{b45364_1s_n0t_3ncrypt10n}`

___

## Double Trouble

**Category**: Cryptography

**Description**: We intercepted a suspicious encoded transmission during routine monitoring. Analysts believe the message was processed through multiple transformation layers before being transmitted. Can you recover the original message? Flag format: `SecLeaf{}`

```shell
cat encrypted2.txt  
526e4a7757584a756333737759544e66655452734d325666616a526d5957  
64664d3245776148523166513d3d0a
```

I concatenate the string and do the cyberchef recipe `from hex` > `from base64` > `rot13`. Cyberchef link: [cyberchef](https://gchq.github.io/CyberChef/#recipe=From_Hex('None')From_Base64('A-Za-z0-9%2B/%3D',true,false)ROT13(true,true,false,13)&input=NTI2ZTRhNzc1NzU4NGE3NTYzMzM3Mzc3NTk1NDRlNjY2NTU0NTI3MzRkMzI1NjY2NjE2YTUyNmQ1OTU3NjQ2NjRkMzI0NTc3NjE0ODUyMzE2NjUxM2QzZDBh)

Or in the terminal:
```shell
echo "526e4a7757584a756333737759544e66655452734d325666616a526d595764664d3245776148523166513d3d0a" | xxd -r -p  
RnJwWXJuc3swYTNfeTRsM2VfajRmYWdfM2EwaHR1fQ==  

echo "RnJwWXJuc3swYTNfeTRsM2VfajRmYWdfM2EwaHR1fQ==" | base64 -d  
FrpYrns{0a3_y4l3e_j4fag_3a0htu}

echo "FrpYrns{0a3_y4l3e_j4fag_3a0htu}" | rot13  
SecLeaf{0n3_l4y3r_w4snt_3n0ugh}
```

I have an alias for `rot13`:
```shell
alias rot13="tr 'A-Za-z' 'N-ZA-Mn-za-m'"
```

`SecLeaf{0n3_l4y3r_w4snt_3n0ugh}`