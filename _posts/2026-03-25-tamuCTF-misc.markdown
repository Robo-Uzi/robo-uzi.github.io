---
layout: post
title:  "Misc Challenges"
date:   2026-03-25 20:19:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /tamuCTF-2026-misc/
---
* TOC
{:toc}

## Quick Response

**Category**: misc

**Author**: `faefeyfa`

**Description**: If you're bored, check this one out!

I get the challenge file:
```shell
file quick-response.png  
quick-response.png: PNG image data, 928 x 928, 2-bit colormap, non-interlaced
```

The image looks like it could be a qr code:

![Alt text](/images/quick-response.png)

After a while I find the entire 29x29 module grid has been XORd with a checkerboard pattern (every module at position where `(row + col) % 2 == 1` is flipped).

In order to solve I need to:
1. Binarize and sample the 928x928 image on a 32px grid > 29x29 module matrix
2. XOR the entire matrix with a checkerboard

Solve script:
```python
#!/usr/bin/env python3
from PIL import Image
import numpy as np

INPUT_FILE = "quick-response.png"
OUTPUT_FILE = "qr_revealed.png"
MODULE_BLOCK = 32  # Each QR module is 32x32 pixels
QR_MODULES = 29    # QR grid is 29x29 modules

# load and process image
im = Image.open(INPUT_FILE).convert("L") # grayscale
data = np.array(im)

binary = (data > 128).astype(np.uint8)

qr_matrix = np.zeros((QR_MODULES, QR_MODULES), dtype=np.uint8)
for row in range(QR_MODULES):
    for col in range(QR_MODULES):
        block = binary[row*MODULE_BLOCK:(row+1)*MODULE_BLOCK,
                       col*MODULE_BLOCK:(col+1)*MODULE_BLOCK]
        qr_matrix[row, col] = int(block.mean() > 0.5)

# apply checkerboard XOR mask to undo masking
for row in range(QR_MODULES):
    for col in range(QR_MODULES):
        if (row + col) % 2 == 1:
            qr_matrix[row, col] ^= 1

# upscale the 29x29 matrix
scale = MODULE_BLOCK
qr_image = Image.fromarray(qr_matrix * 255).resize(
    (QR_MODULES*scale, QR_MODULES*scale), resample=Image.NEAREST
)

# save revealed QR
qr_image.save(OUTPUT_FILE)
qr_image.show()
print(f"QR code saved to {OUTPUT_FILE}")
```

Output image:

![Alt text](/images/qr_revealed.png)

This is a readable qr code which holds the flag!

`gigem{d1d_y0u_n0t1c3_th3_t1m1n9_b175}`

___

## Short Term Fuel Trim

**Category**: misc

**Author**: `beds`

**Description**: Some stupid numbers or something. Not really sure what they're for, maybe you can figure it out? NOTE: The flag is fully lowercase! HINT: the flag contains the letter 'i'

I get the challenge file:
```shell
cat numbers.txt | wc -l  
178021  

cat numbers.txt | head  
# STFT shape: complex64 (129, 1380)  
(0.000000000000000000e+00+0.000000000000000000e+00j)  
(0.000000000000000000e+00+0.000000000000000000e+00j)  
(-1.484999775886535645e+00+0.000000000000000000e+00j)  
(2.873720169067382812e+00+0.000000000000000000e+00j)  
(9.447572708129882812e+00+0.000000000000000000e+00j)  
(8.104647827148437500e+01+0.000000000000000000e+00j)  
(1.316259765625000000e+01+0.000000000000000000e+00j)  
(-3.101673889160156250e+02+0.000000000000000000e+00j)  
(-4.186026916503906250e+02+0.000000000000000000e+00j)  

cat numbers.txt | tail  
(0.000000000000000000e+00+0.000000000000000000e+00j)  
(0.000000000000000000e+00+0.000000000000000000e+00j)  
(0.000000000000000000e+00+0.000000000000000000e+00j)  
(0.000000000000000000e+00+0.000000000000000000e+00j)  
(0.000000000000000000e+00+0.000000000000000000e+00j)  
(0.000000000000000000e+00+0.000000000000000000e+00j)  
(0.000000000000000000e+00+0.000000000000000000e+00j)  
(0.000000000000000000e+00+0.000000000000000000e+00j)  
(0.000000000000000000e+00+0.000000000000000000e+00j)  
(0.000000000000000000e+00+0.000000000000000000e+00j)
```

`numbers.txt` stores a 129x1380 STFT ([Short-time Fourier transform](https://en.wikipedia.org/wiki/Short-time_Fourier_transform)) matrix (complex64). 

With this python I reconstruct the audio:
```python
import numpy as np
import re
from scipy.signal import istft
import wave, struct

with open('numbers.txt', 'r') as f:
    lines = f.readlines()

shape_match = re.search(r'\((\d+),\s*(\d+)\)', lines[0])
rows, cols = int(shape_match.group(1)), int(shape_match.group(2))

data = []
for line in lines[1:]:
    line = line.strip()
    if line:
        c = complex(line.replace('(','').replace(')',''))
        data.append(c)

stft_matrix = np.array(data, dtype=np.complex64).reshape(rows, cols)

# reconstruct audio with istft
# 129 freq bins = nperseg of 256 (256/2+1=129)
nperseg = 256
_, audio = istft(stft_matrix, nperseg=nperseg, noverlap=nperseg//2)
print(f"Audio length: {len(audio)} samples")

# save as WAV (assume 44100 Hz sample rate)
audio_norm = audio / np.max(np.abs(audio) + 1e-9)
audio_int16 = (audio_norm * 32767).astype(np.int16)

with wave.open('output.wav', 'w') as wf:
    wf.setnchannels(1)
    wf.setsampwidth(2)
    wf.setframerate(44100)
    wf.writeframes(audio_int16.tobytes())
print("Saved output.wav")
```

From listening to the audio slowed down I hear the flag read out loud.

`gigem{fft_is_50_0p}`

___

## pittrap

**Category**: misc

**Author**: `flocto`

**Description**: don't fall for the oldest trick in the book

I get the challenge file:
```shell
file gigem.onnx  
gigem.onnx: data

strings -n 10 gigem.onnx | head  
2.10.0+cu128:  
input_ids  
node_Shape_0"  
namespace  
v_empty_nn_module_stack_from_metadata_hook: _empty_nn_module_stack_from_metadata_hook/sym_size_int_1: aten.sym_size.intJd  
pkg.torch.onnx.class_hierarchy  
B['_empty_nn_module_stack_from_metadata_hook', 'aten.sym_size.int']J  
pkg.torch.onnx.fx_node  
p%sym_size_int_1 : [num_users=1] = call_function[target=torch.ops.aten.sym_size.int](args = (%x, 0), kwargs = {})J] 
pkg.torch.onnx.name_scopes
```

`gigem.onnx` is a open neural network exchange (ONNX) model. ONNX is a portable format used to store neural networks so they can run in frameworks like PyTorch. The model is basically a flag checker.

From `strings` I can extract these weights:
```shell
embed.weight
conv.weight
conv.bias
fc1.weight
fc1.bias
fc2.weight
fc2.bias
```

The input takes 48 ASCII tokens and the output is a single float score. Most inputs produce the same constant output. 

Solve script:
```python
import onnx
import numpy as np
from onnx import numpy_helper

model = onnx.load("gigem.onnx")
weights = {init.name: numpy_helper.to_array(init) for init in model.graph.initializer}

embed_w = weights['embed.weight']
conv_w = weights['conv.weight']
conv_b = weights['conv.bias']
fc1_w = weights['fc1.weight']
fc1_b = weights['fc1.bias']
fc2_w = weights['fc2.weight'][0]
fc2_b = weights['fc2.bias'][0]

def gelu(x):
    return 0.5 * x * (1 + np.tanh(np.sqrt(2/np.pi) * (x + 0.044715 * x**3)))

def forward_batched(tokens_batch):
    B = tokens_batch.shape[0]
    embed = embed_w[tokens_batch]
    perm = embed.transpose(0, 2, 1)
    padded = np.pad(perm, ((0,0),(0,0),(2,2)))
    out_len = 48
    conv = (np.tensordot(padded[:, :, :out_len], conv_w[:, :, 0], axes=([1],[1])).transpose(0,2,1) +
            np.tensordot(padded[:, :, 2:out_len+2], conv_w[:, :, 1], axes=([1],[1])).transpose(0,2,1) +
            np.tensordot(padded[:, :, 4:out_len+4], conv_w[:, :, 2], axes=([1],[1])).transpose(0,2,1) +
            conv_b[np.newaxis, :, np.newaxis])
    g = gelu(conv)
    view = g.reshape(B, -1)
    lin = view @ fc1_w.T + fc1_b
    g1 = gelu(lin)
    return (g1 @ fc2_w) + fc2_b

def score_single(tokens):
    return forward_batched(np.array(tokens, dtype=np.int64)[np.newaxis])[0]

# Restricted charset: lowercase letters + underscore + digits
flag_inner_chars = [ord(c) for c in 'abcdefghijklmnopqrstuvwxyz_0123456789']

base = list(b'gigem{just_c') + [ord('a')]*30 + list(b'_fine}')
tokens = np.array(base, dtype=np.int64)

np.random.seed(42)
best_score = -999
best_tokens = tokens.copy()

for restart in range(40):
    if restart == 0:
        tokens = np.array(base, dtype=np.int64)
    else:
        tokens = best_tokens.copy()
        n = np.random.randint(1, 10)
        positions = np.random.choice(range(12, 42), n, replace=False)
        tokens[positions] = np.random.choice(flag_inner_chars, n)
    
    for iteration in range(40):
        improved = False
        order = np.random.permutation(range(12, 42))
        for pos in order:
            batch = np.tile(tokens, (256, 1))
            batch[:, pos] = np.arange(256, dtype=np.int64)
            scores = forward_batched(batch)
            # restrict to our charset
            mask = np.full(256, -9999.0)
            mask[flag_inner_chars] = scores[flag_inner_chars]
            best_t = int(np.argmax(mask))
            if best_t != tokens[pos]:
                improved = True
                tokens[pos] = best_t
        if not improved:
            break
    
    s = score_single(tokens)
    if s > best_score:
        best_score = s
        best_tokens = tokens.copy()
        flag = ''.join(chr(t) for t in tokens)
        print(f"Restart {restart}: {s:.4f} -> {flag}", flush=True)

print(f"\nFINAL: {''.join(chr(t) for t in best_tokens)}")
print(f"Score: {best_score:.4f}")
```

Output:
```shell
python3 solve.py  
Restart 0: 98.4594 -> gigem{just_catch_the_local_max_and_ubll_be_fine}
```
`'` is not in the character set but `ubll` should be changed to `u'll` and that is the flag.

`gigem{just_catch_the_local_max_and_u'll_be_fine}`
