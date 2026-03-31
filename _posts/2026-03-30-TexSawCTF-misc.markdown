---
layout: post
title:  "Misc Challenges"
date:   2026-03-30 20:37:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /TexSawCTF-2026-misc/
---
* TOC
{:toc}

## Excellent Neurons

**Author**: Written by Auric115

**Category**: misc

**Description**: Find the flag by reverse engineering this neural network. Oh, and its in Excel. Flag format: `texsaw{flag}` ex: `texsaw{orthogonal}`

I get the challenge file:
```shell
file challenge.xlsx  
challenge.xlsx: Microsoft Excel 2007+
```

When I open the file I find parameters layed out for a neural network. Looking at it:
- Input layer: 30 neurons (each takes `(ASCII value) / 127`)
- Hidden Layer 1: 60 neurons with ReLU activation
- Hidden Layer 2: 1 neuron with ReLU
- Output laye: 4 neurons with sigmoid (F, L, A, G)

Each hidden neuron is connected to exactly one input with either `+1` or `-1` weight. The bias values will give away the flag. 

Solve script:
```python
import numpy as np

biases = {
    0: 0.9133858,   # from h1[23]
    1: 0.795276,
    2: 0.944882,
    3: 0.905512,
    4: 0.76378,
    5: 0.937008,
    6: 0.968504,
    7: 0.866142,
    8: 0.401575,
    9: 0.92126,
    10: 0.89764,
    11: 0.40945,
    12: 0.85039,
    13: 0.74803,
    14: 0.89764,
    15: 0.401575,
    16: 0.929134,
    17: 0.401575,
    18: 0.897638,
    19: 0.905512,
    20: 0.401575,
    21: 0.984252,
}

# Convert to ASCII
flag_chars = []
for i in range(22):
    if i in biases:
        ascii_val = int(round(biases[i] * 127))
        flag_chars.append(chr(ascii_val))
    else:
        flag_chars.append(chr(0))
flag = ''.join(flag_chars)
print(flag)
```

The network acts like a pattern matcher where each flag character is encoded in the biases of the first layer. By converting these bias values back to ASCII (multiplying by 127 and rounding) you can recover the exact input that makes the network output `FLAG` instead of `FAIL`.

`texsaw{n3ur4l_r3v3rs3}`

___

## Secret Word 🤐

**Author**: Written by Auric115

**Category**: misc

**Description**: Find the flag hidden in this suspicious Word Document. Flag format: `texsaw{flag}` ex: `texsaw{orthogonal}`

I get the challenge file, then unzip it:
```shell
file challenge.docx  
challenge.docx: Microsoft Word 2007+

unzip challenge.docx  
Archive:  challenge.docx  
inflating: word/document.xml  
inflating: word/settings.xml  
inflating: docProps/custom.xml  
inflating: [Content_Types].xml  
inflating: docProps/app.xml  
inflating: docProps/core.xml  
inflating: word/_rels/document.xml.rels  
inflating: word/_rels/numbering.xml.rels  
inflating: word/fontTable.xml  
inflating: word/numbering.xml  
inflating: word/styles.xml  
inflating: word/media/image1.png  
inflating: word/media/image2.jpeg  
inflating: word/media/image3.jpeg  
inflating: word/media/image4.jpeg  
inflating: word/media/image5.jpeg  
inflating: word/media/image6.jpeg  
inflating: word/media/image7.jpeg  
inflating: word/media/image8.png  
inflating: word/media/image9.png  
inflating: word/media/image10.png  
... etc ...
inflating: word/media/image82.png  
inflating: word/media/image83.png  
inflating: word/media/image84.png  
inflating: word/media/image85.png  
inflating: word/media/image86.png  
inflating: word/media/image87.png  
inflating: word/media/image88.png  
inflating: word/media/image89.png  
inflating: word/media/image90.png  
inflating: word/media/image91.png  
inflating: word/media/image92.png  
inflating: word/media/image93.png  
inflating: word/theme/theme1.xml  
inflating: _rels/.rels  
extracting: secret.txt

cat secret.txt  
dGV4c2F3e3N1cnByMXNlIV93MHJkX2YxbGVzX2FyM196MXBfNHJjaGl2ZXNfNjA3MDkwMTM3NzF9

echo "dGV4c2F3e3N1cnByMXNlIV93MHJkX2YxbGVzX2FyM196MXBfNHJjaGl2ZXNfNjA3MDkwMTM3NzF9" | base64 -d  
texsaw{surpr1se!_w0rd_f1les_ar3_z1p_4rchives_60709013771}
```

`texsaw{surpr1se!_w0rd_f1les_ar3_z1p_4rchives_60709013771}`
