---
layout: post
title:  "Forensics Challenges"
date:   2026-08-11 20:49:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /script-CTF-2026-forensics/
---
* TOC
{:toc}

## Bruteforced

**Category**: forensics

**Description**: Help! Our website got bruteforced. Hopefully the attacker did not leak anything.

I download the challenge file:
```shell
file log.pcap  
log.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 262144)
```

I opened the pcap in wireshark. The pcap contained `100,004` packets. The attacker brute forced from `/flag_1` to `/flag_9999`. 

By searching for `http.response.code == 200` I found a successful request:
```http
GET /flag_4919 HTTP/1.1
Host: ctf.scriptsorcerers.xyz
User-Agent: python-requests/2.32.3
Accept-Encoding: gzip, deflate, br
Accept: */*
Connection: keep-alive
```

Response:
```http
HTTP/1.1 200 OK
Server: Werkzeug/2.2.2 Python/3.13.12
Date: Tue, 14 Jul 2026 11:18:41 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 0
Connection: close
```

I went to `https://ctf.scriptsorcerers.xyz/flag_4919` to get the flag.

`scriptCTF{7h3_h1dd3n_3ndp01n7_g0t_l34k3d}`

___

## RecoverMyPet

**Category**: forensics

**Author**: Connor Chang

**Description**: this is all i have left.

I get the challenge files:
```shell
unzip images.zip
Archive:  images.zip
 extracting: 11_17.png
  inflating: 11_7.png
 extracting: 13_19.png
 extracting: 13_9.png
 extracting: 17_11.png
 extracting: 17_23.png
  inflating: 19_13.png
  inflating: 19_29.png
 extracting: 1_1.png
 extracting: 1_2.png
 extracting: 23_17.png
  inflating: 23_31.png
  inflating: 29_19.png
 extracting: 29_37.png
  inflating: 2_1.png
 extracting: 2_3.png
 extracting: 31_23.png
 extracting: 31_41.png
  inflating: 37_29.png
 extracting: 37_43.png
 extracting: 3_2.png
  inflating: 3_5.png
  inflating: 41_31.png
 extracting: 41_59.png
 extracting: 43_37.png
  inflating: 4_7.png
 extracting: 53_61.png
 extracting: 59_41.png
  inflating: 5_3.png
 extracting: 5_8.png
 extracting: 61_53.png
 extracting: 73_97.png
 extracting: 7_11.png
 extracting: 7_4.png
  inflating: 8_5.png
  inflating: 9_13.png
```

There are 36 images. Most of the images look very distorted but a few of them are clear:

![Alt text](/images/scriptimages.png)

Sample clear image:

![Alt text](/images/41_59.png)

Sample distorted image:

![Alt text](/images/29_19.png)

The distorted images contained a hint in the metadata:
```shell
exiftool 2_1.png | grep Description  
Image Description               : i once had a cat, u know. but this crazy scientist took my cat and turned it into a donut 150 times and i cant find my cat anymore! (4/4)
```
This references [Arnolds Cat Map](https://en.wikipedia.org/wiki/Arnold%27s_cat_map). `(4/4)` represents the coordinates for the tile. Since there are 36 images total, the final image is a 6 by 6 grid.

Each image contained a different number:
```shell
exiftool 37_29.png | grep Description  
Image Description               : i once had a cat, u know. but this crazy scientist took my cat and turned it into a donut 5 times and i cant find my cat anymore! (6/3)
```

The filename (`2_1.png`) correspond to the `p` and `q` variables in a `Arnolds Cat Map matrix`. To recover the image I can iterate over the coordinates of a blank canvas, apply the Arnolds Cat Map formula forward `n` times to find where each pixel was relocated in the distorted image, and map the colors back.

I ran this script to recover each image:
```python
import os
import re
from PIL import Image

def unscramble():
    tile_size = 60
    
    # Create the output directory if it doesn't exist
    output_dir = "test_output"
    os.makedirs(output_dir, exist_ok=True)
    
    for filename in os.listdir("."):
        if not filename.endswith(".png") or "_" not in filename:
            continue
            
        # parse p and q from filename (11_17.png > p=11, q=17)
        base = filename.replace(".png", "")
        p, q = map(int, base.split("_"))
        
        # get iterations and position from the metadata
        img = Image.open(filename)
        desc = img.getexif().get(270) # 270 is the EXIF tag ID for ImageDescription
        
        if not desc:
            desc = img.info.get("Description", "")
            
        # regex to pull out the numbers (donut 150 times ... (4/4))
        m = re.search(r"(\d+) times.*\((\d+)/(\d+)\)", desc)
        if not m:
            continue
            
        iterations = int(m.group(1))
        # Keep the raw strings for renaming the output files
        grid_x_str = m.group(2)
        grid_y_str = m.group(3)
        
        # unscramble the tile
        N = tile_size
        recovered = Image.new("RGB", (N, N))
        pixels_in = img.load()
        pixels_out = recovered.load()
        
        for x in range(N):
            for y in range(N):
                curr_x, curr_y = x, y
                for _ in range(iterations):
                    next_x = (curr_x + p * curr_y) % N
                    next_y = (q * curr_x + (p * q + 1) * curr_y) % N
                    curr_x, curr_y = next_x, next_y
                pixels_out[x, y] = pixels_in[curr_x, curr_y]
                
        # save the decrypted tile into the test_output folder
        out_filename = f"pos_{grid_x_str}_{grid_y_str}_{filename}"
        out_path = os.path.join(output_dir, out_filename)
        
        recovered.save(out_path)
        print(f"Decrypted {filename} > Saved to {out_path}")

    print(f"\nAll decrypted tiles are in the '{output_dir}' folder.")

if __name__ == "__main__":
    unscramble()
```

Once I had each normal image in the `test_output` directory I ran this script to reassemble the image. I did find a website that would simply assemble the 6 by 6 images for me. However my browser was corrupting the image or something so I did the annoying/more fun thing and made this script:
```python
#!/usr/bin/env python3
import os
import re
from PIL import Image

def main():
    folder = "test_output"
    pattern = re.compile(r"pos_(\d+)_(\d+)_")
    
    tiles = {}
    tile_size = None

    for fname in os.listdir(folder):
        if not fname.endswith(".png"):
            continue
        match = pattern.match(fname)
        if not match:
            print(f"Skipping {fname}: filename does not match pattern")
            continue
        
        row = int(match.group(1)) - 1
        col = int(match.group(2)) - 1
        if row >= 6 or col >= 6:
            print(f"Skipping {fname}: row/col out of bounds ({row+1},{col+1})")
            continue
        
        path = os.path.join(folder, fname)
        img = Image.open(path).convert("RGB")
        
        if tile_size is None:
            tile_size = img.size
        elif img.size != tile_size:
            print(f"Warning: {fname} has different size {img.size}, resizing to {tile_size}")
            img = img.resize(tile_size)
        
        tiles[(row, col)] = img
    
    if tile_size is None:
        print("No valid tiles found!")
        return
    w, h = tile_size
    canvas = Image.new("RGB", (w * 6, h * 6), color=(255,255,255))
    
    for (row, col), img in tiles.items():
        canvas.paste(img, (col * w, row * h))
    
    canvas.save("flag.png")
    print(f"Saved as flag.png (size: {canvas.size})")

if __name__ == "__main__":
    main()
```

Final image:

![Alt text](/images/flag75867835672.png)

`scriptCTF{w@t_4_cu71e_p@too1$}`
