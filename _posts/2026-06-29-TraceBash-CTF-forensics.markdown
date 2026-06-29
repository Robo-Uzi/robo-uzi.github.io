---
layout: post
title:  "Forensics Challenges"
date:   2026-06-29 14:10:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /TraceBash-CTF-2026-forensics/
---

**Title**: Bespoke Superblock

**Category**: forensics

**Author:** CYB3RFY

**Description**: We recovered a strange disk image from a compromised server. Standard forensic tools are having trouble making sense of its contents, though a partially corrupted extraction script was found nearby. Can you investigate the image and piece together the hidden information?

I get the challenge files:
```shell
unzip bespoke_superblock.zip  
Archive:  bespoke_superblock.zip  
  inflating: challenge.img  
  inflating: parser.py
```

`challenge.img`:
```shell
file challenge.img  
challenge.img: DOS executable (COM), start instruction 0xeb3c904d 53444f53
```

`parser.py`:
```python
import struct
import sys

def recover_flag(img_path):
    with open(img_path, "rb") as f:
        # Seek to custom filesystem start
        f.seek(0x1000)
        header = f.read(16)
        
        if len(header) < 16:
            print("[-] Error: Image too small.")
            return

        # Unpack Superblock: Magic (4s), BlockSize (H), TotalBlocks (I), FlagInode (I)
        magic, block_size, total_blocks, flag_inode = struct.unpack('<4s H I I 2x', header)
        
        if magic != b'TBFS':
            print(f"[-] Error: Invalid Magic {magic}. Are you at 0x1000?")
            return

        print(f"[+] Found Custom Filesystem!")
        print(f"    Block Size: {block_size}")
        print(f"    Total Blocks: {total_blocks}")
        print(f"    Flag Inode: 0x{flag_inode:X}")
        
        print("\n[+] Attempting Flag Recovery...")
        recovered_data = b""
        
        for i in range(total_blocks):
            # Calculate block offset
            offset = flag_inode + (i * block_size)
            f.seek(offset)
            
            # Read first 4 bytes of the block (the flag chunk)
            chunk = f.read(4)
            
            # TODO: We observed some high entropy in the data. 
            # Could there be an additional encoding layer?
            # For now, just appending raw bytes.
            recovered_data += chunk
            
        print(f"\n[!] Recovered Data: {recovered_data}")
        print("[!] Warning: Data appears corrupted. Check spatial consistency.")

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python3 parser.py <challenge.img>")
    else:
        recover_flag(sys.argv[1])
```

Running `parser.py`:
```shell
python3 parser.py challenge.img  
[+] Found Custom Filesystem!  
    Block Size: 512  
	Total Blocks: 8  
	Flag Inode: 0x1020  
  
[+] Attempting Flag Recovery...  
  
[!] Recovered Data: b'tbctf[SPAT\x11AL\x7fAWARE\x7fXOR\x7f\x11\x13\x13\x17]   '  
[!] Warning: Data appears corrupted. Check spatial consistency.
```

XOR each byte with `0x20` and swap the brackets:
```python
python3  
Python 3.13.5 (main, May 5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> raw = b'tbctf[SPAT\x11AL\x7fAWARE\x7fXOR\x7f\x11\x13\x13\x17]   '  
... fixed = bytes(b ^ 0x20 for b in raw).replace(b'[', b'{').replace(b']', b'}').strip()  
... print(fixed.decode())  
...  
TBCTF{spat1al_aware_xor_1337}
```

`TBCTF{spat1al_aware_xor_1337}`
