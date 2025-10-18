---
layout: post
title:  "Baby_Reverse_Revenge_From_NHNC"
date:   2025-10-18 15:57:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-baby-reverse-revenge-from-NHNC/
---

**Title:** Baby_Reverse_Revenge_From_NHNC

**Category:** reverse

**Description:** I make a reverse challenge from NHNC 2025 but I think it is too easy so I make a revenge version.

**Author:** LemonTea

I get two challenge files:
```shell
file encrypter  
encrypter: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=752a80f792346585e1f82a8489545681ceff877d, for GNU/Linux 4.4.0, not stripped

cat flag.enc  
�F���W�M�Z�EA��_�ݷ;Ѝ��RR�e1�٘@�@��]��8sN
```

I open the binary in ghidra. Here are 4 functions which I think are important (`main`, `do_crypto`, `encrypt_file`, and `call_embedded_shellcode`):
```d
undefined8 main(int param_1,undefined8 *param_2)

{
  int iVar1;
  undefined8 uVar2;
  long in_FS_OFFSET;
  undefined8 local_48;
  undefined8 local_40;
  undefined8 local_38;
  undefined8 local_30;
  undefined8 local_28;
  undefined8 local_20;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  if (param_1 < 2) {
    printf("Usage: %s encrypt",*param_2);
    uVar2 = 1;
  }
  else {
    local_48 = 0;
    local_40 = 0;
    strncpy((char *)&local_48,"1337",0x10);
    local_38 = 0;
    local_30 = 0;
    local_28 = 0;
    local_20 = 0;
    iVar1 = call_embedded_shellcode(&local_38,0x20);
    if (iVar1 == 0) {
      fwrite("Failed to produce key via shellcode\n",1,0x24,stderr);
      uVar2 = 2;
    }
    else {
      iVar1 = strcmp((char *)param_2[1],"encrypt");
      if (iVar1 == 0) {
        iVar1 = encrypt_file("flag.txt","flag.enc",&local_38,&local_48);
        if (iVar1 == 0) {
          puts("Encrypt failed");
          uVar2 = 3;
        }
        else {
          puts("Encrypted -> flag.enc");
          uVar2 = 0;
        }
      }
      else {
        uVar2 = 0;
      }
    }
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return uVar2;
}
```

```d
undefined8 do_crypto(FILE *param_1,FILE *param_2,uchar *param_3,uchar *param_4,int param_5)

{
  int iVar1;
  undefined8 uVar2;
  size_t sVar3;
  long in_FS_OFFSET;
  int local_850;
  int local_84c;
  EVP_CIPHER_CTX *local_848;
  EVP_CIPHER *local_840;
  uchar local_838 [1024];
  uchar local_438 [1064];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_848 = EVP_CIPHER_CTX_new();
  if (local_848 == (EVP_CIPHER_CTX *)0x0) {
    uVar2 = 0;
  }
  else {
    local_840 = EVP_aes_256_cbc();
    if (param_5 == 0) {
      iVar1 = EVP_DecryptInit_ex(local_848,local_840,(ENGINE *)0x0,param_3,param_4);
      if (iVar1 != 1) {
        handle_errors();
      }
    }
    else {
      iVar1 = EVP_EncryptInit_ex(local_848,local_840,(ENGINE *)0x0,param_3,param_4);
      if (iVar1 != 1) {
        handle_errors();
      }
    }
    while( true ) {
      sVar3 = fread(local_838,1,0x400,param_1);
      local_84c = (int)sVar3;
      if (local_84c < 1) break;
      if (param_5 == 0) {
        iVar1 = EVP_DecryptUpdate(local_848,local_438,&local_850,local_838,local_84c);
        if (iVar1 != 1) {
          handle_errors();
        }
      }
      else {
        iVar1 = EVP_EncryptUpdate(local_848,local_438,&local_850,local_838,local_84c);
        if (iVar1 != 1) {
          handle_errors();
        }
      }
      fwrite(local_438,1,(long)local_850,param_2);
    }
    if (param_5 == 0) {
      iVar1 = EVP_DecryptFinal_ex(local_848,local_438,&local_850);
      if (iVar1 != 1) {
        handle_errors();
      }
    }
    else {
      iVar1 = EVP_EncryptFinal_ex(local_848,local_438,&local_850);
      if (iVar1 != 1) {
        handle_errors();
      }
    }
    fwrite(local_438,1,(long)local_850,param_2);
    EVP_CIPHER_CTX_free(local_848);
    uVar2 = 1;
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return uVar2;
}
```

```d
undefined4 encrypt_file(char *param_1,char *param_2,undefined8 param_3,undefined8 param_4)

{
  undefined4 uVar1;
  FILE *__stream;
  FILE *__stream_00;
  
  __stream = fopen(param_1,"rb");
  __stream_00 = fopen(param_2,"wb");
  if ((__stream == (FILE *)0x0) || (__stream_00 == (FILE *)0x0)) {
    perror("File open");
    if (__stream != (FILE *)0x0) {
      fclose(__stream);
    }
    if (__stream_00 != (FILE *)0x0) {
      fclose(__stream_00);
    }
    uVar1 = 0;
  }
  else {
    uVar1 = do_crypto(__stream,__stream_00,param_3,param_4,1);
    fclose(__stream);
    fclose(__stream_00);
  }
  return uVar1;
}
```

```d
undefined8 call_embedded_shellcode(undefined8 param_1,undefined8 param_2)

{
  int iVar1;
  ulong uVar2;
  size_t __len;
  code *__dest;
  undefined8 uVar3;
  
  uVar2 = sysconf(0x1e);
  __len = uVar2 * ((uVar2 + 0x80) / uVar2);
  __dest = (code *)mmap((void *)0x0,__len,7,0x22,-1,0);
  if (__dest == (code *)0xffffffffffffffff) {
    perror("mmap");
    uVar3 = 0;
  }
  else {
    memcpy(__dest,shellcode,0x81);
    (*__dest)(param_1,param_2);
    iVar1 = munmap(__dest,__len);
    if (iVar1 != 0) {
      perror("munmap");
    }
    uVar3 = 1;
  }
  return uVar3;
}
```
Looking at the shellcode in ghidra I can see it uses repeated instructions (opcode `C6 47 <disp8> <imm8>`) to write individual bytes into the buffer at runtime.

I make a solve script which scans the `encrypter` binary for `C6 47` occurrences and reads the following two bytes as `(disp8, imm8)`. It places `imm8` at `key[disp8]`, and when it has at least 32 bytes, constructs the AES‑256 key. 

It then tries to decrypt `flag.enc` using IV `b"1337" + b"\x00"*12` (exactly what the program uses) and prints the recovered key and any possible flag.

Solve script:
```python
#!/usr/bin/env python3
import os, sys
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

BIN = "encrypter"
CIPH = "flag.enc"
IV = b"1337" + b"\x00" * 12  # program uses strncpy(..., "1337", 0x10)

def extract_key_bytes_from_binary(binpath):
    data = open(binpath,"rb").read()
    keybytes = {}
    i = 0
    while True:
        idx = data.find(b'\xC6\x47', i)
        if idx == -1:
            break
        # ensure there are two bytes after opcode
        if idx + 4 <= len(data):
            disp = data[idx+2]
            imm  = data[idx+3]
            # store imm at displacement disp
            keybytes[disp] = imm
        i = idx + 1
    return keybytes

def try_decrypt_with_key(key_bytes, ciphertext):
    key = bytes(key_bytes)
    cipher = AES.new(key, AES.MODE_CBC, IV)
    pt = cipher.decrypt(ciphertext)
    # try unpad (if valid PKCS#7)
    try:
        pt2 = unpad(pt, AES.block_size)
        return pt2
    except ValueError:
        # return raw decrypted data if it looks printable
        return pt

def looks_like_flag_text(pt: bytes):
    try:
        s = pt.decode('utf-8', errors='ignore')
    except:
        return False
    return ("QnQSec{" in s) or ("{" in s and "}" in s) or any(x in s for x in ("QnQ"))

def main():
    if not (os.path.isfile(BIN) and os.path.isfile(CIPH)):
        print("Check your directory.")
        sys.exit(1)

    keydict = extract_key_bytes_from_binary(BIN)
    if not keydict:
        print("No C6 47 patterns found!? :(")
        sys.exit(1)

    # build a 32 byte key using collected displacements
    key_array = [0]*32
    for d,v in keydict.items():
        if d < 32:
            key_array[d] = v

    key_bytes = bytes(key_array)
    print("Recovered key bytes (hex):", key_bytes.hex())
    try:
        print("Recovered key (utf-8):", key_bytes.decode('utf-8', errors='replace'))
    except:
        pass

    ct = open(CIPH,"rb").read()
    pt = try_decrypt_with_key(key_bytes, ct)

    if looks_like_flag_text(pt):
        try:
            print("Flag:", pt.decode('utf-8', errors='replace'))
        except:
            print(pt)
    else:
        print("\nDecrypted output doesn't look like a flag:")
        try:
            s = pt.decode('utf-8', errors='replace')
            print(s[:400])
        except:
            print(pt[:200].hex())

if __name__ == "__main__":
    main()
```

Output:
```shell
python3 solve5.py  
Recovered key bytes (hex): 7468315f31735f7468335f76616c75335f30665f6b3379000000000000000000  
Recovered key (utf-8): th1_1s_th3_valu3_0f_k3y  
Flag: QnQSec{a_s1mpl3_fil3_3ncrypt3d_r3v3rs3}
```

`QnQSec{a_s1mpl3_fil3_3ncrypt3d_r3v3rs3}`