---
layout: post
title:  "Misc Challenges"
date:   2026-08-04 17:35:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /BashBash-CTF-misc/
---
* TOC
{:toc}
## Spiritual Interception

**Category**: misc

**Author**: Vals

**Description**: We've received an odd transmission that appears to be largely made of noise. However, we think there may be more to this audio than meets the eye...

I download the challenge file `transmission.wav`:
```shell
file transmission.wav  
transmission.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 44100 Hz
```

I opened the file in audacity and looked at the spectrogram view:

![Alt text](/images/spectrogram3896582.png)

`bushbash{s33ing-gh0sts}`

___

## The CSSA Hackerman I

**Category**: OSINT

**Author**: Cameron & Vals

**Description**: The CSSA Hackerman is on the run! After successfully hacking into the CSSA Mainframe™, the Hackerman is now making a desperate escape from the CSSA Common Room.

Soon, the Hackerman will be dashing through the suburbs of Canberra, and we need your help hunting them down.

The flag is the coordinates where the Hackerman is standing, to four decimal places. For example, accepted flags would be `bushbash{-35.3081,149.1244}` or `bushbash{-35.3082,149.1244}` if the Hackerman were standing underneath the Australian Parliament House flagpole.

I get the challenge file:

![Alt text](/images/aba45f2022fa2a4f28ca87b2cf1a1436.jpeg)

[google maps link](https://www.google.com/maps/place/35%C2%B016'31.4%22S+149%C2%B007'14.5%22E/@-35.2756326,149.1211157,565a,30.5y,329.89h,94.09t/data=!3m10!1e1!3m8!1sR5QVGPdSTJJ-MhZoICEZmQ!2e0!6shttps:%2F%2Fstreetviewpixels-pa.googleapis.com%2Fv1%2Fthumbnail%3Fcb_client%3Dmaps_sv.tactile%26w%3D900%26h%3D600%26pitch%3D-4.090059365553728%26panoid%3DR5QVGPdSTJJ-MhZoICEZmQ%26yaw%3D329.89427713249006!7i16384!8i8192!9m2!1b1!2i46!4m4!3m3!8m2!3d-35.2754!4d149.1207?entry=ttu&g_ep=EgoyMDI2MDcyOS4wIKXMDSoASAFQAw%3D%3D)

`bushbash{-35.2754,149.1207}`

___

## The CSSA Hackerman II

**Category**: OSINT

**Author**: Cameron & Vals

**Description**: After making a daring escape from the common room, the CSSA Hackerman hopped into his getaway car, which screamed away at 88mph.

The CSSA, of course, follows the speed limit at all times, so the getaway car got a pretty good head start. Confident that we're off their tail, the Hackerman is taking a rest stop by a lake.

The flag is the coordinates where the Hackerman is resting, to four decimal places. For example, accepted flags would be `bushbash{-35.3081,149.1244}` or `bushbash{-35.3082,149.1244}` if the Hackerman were standing underneath the Australian Parliament House flagpole.

I get the challenge image:

![Alt text](/images/b35da867a03e793b0d7077933fb45c15.jpeg)

I cropped the image to search for the skyline:

![Alt text](/images/bushbashosintcropped.png)

I put the cropped image into [https://reverseimagesearcher.com/](https://reverseimagesearcher.com/) and found the location:

![Alt text](/images/bushbashosint1.png)

[google maps link](https://www.google.com/maps/place/35%C2%B014'08.0%22S+149%C2%B004'22.6%22E/@-35.2361554,149.0714158,1403m/data=!3m1!1e3!4m4!3m3!8m2!3d-35.235542!4d149.07295?entry=ttu&g_ep=EgoyMDI2MDcyOS4wIKXMDSoASAFQAw%3D%3D)

[openstreetmap](https://www.openstreetmap.org/search?lat=-35.235542&lon=149.072950&zoom=17#map=17/-35.235538/149.072950)

`bushbash{-35.2355,149.0729}`

___

## Hack The Vault I

**Category**: rev

**Author**: Harold Gao

**Description**: The jungle holds many secrets, some of them as dark as the night ruled by a laughing moon. Ever since the Moss Man committed his atrocious acts, the villagers slept with an eye open, while detective Kane searches for the taunting vaults he left behind. He wants to talk to you, he needs your help: `nc 34.40.133.67 7776`.

I get the challenge file:
```shell
file vault  
vault: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6e8f60893ec6dec56766f62933c5d6f50620cb2d, for GNU/Linux 3.2.0, stripped
```

I put the binary in ghidra and found these functions:
```c
void FUN_001011c9(void)

{
  long lVar1;
  long in_FS_OFFSET;
  
  lVar1 = *(long *)(in_FS_OFFSET + 0x28);
  printf(PTR_s___________________________________00104060);
  putchar(10);
  puts("Hey. Detective Kane here. I found a vault, right by the river burried two feet underground."
      );
  puts("I dug it up, seems like I need a password. Do you know what it is?");
  if (lVar1 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}
```

```c
undefined8 FUN_00101233(void)

{
  int iVar1;
  size_t sVar2;
  undefined8 uVar3;
  long in_FS_OFFSET;
  size_t local_128;
  undefined8 local_120;
  char local_118 [264];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  printf("Enter the password: ");
  fgets(local_118,0x100,stdin);
  local_120 = strlen(PTR_s_th3M0ssM4ni5h3re,y0uc4ntcatchm3_00104068);
  sVar2 = strlen(local_118);
  local_128 = sVar2;
  if ((sVar2 != 0) && (local_118[sVar2 - 1] == '\n')) {
    local_128 = sVar2 - 1;
    local_118[sVar2 - 1] = '\0';
  }
  if (local_120 == local_128) {
    iVar1 = strncmp(local_118,PTR_s_th3M0ssM4ni5h3re,y0uc4ntcatchm3_00104068,local_128);
    if (iVar1 == 0) {
      uVar3 = 1;
    }
    else {
      uVar3 = 0;
    }
  }
  else {
    uVar3 = 0;
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return uVar3;
}
```

I send the password `th3M0ssM4ni5h3re,y0uc4ntcatchm3` to the server:
```shell
nc 34.40.133.67 7776  
__ __   ____    __  __  _      ______  __ __    ___      __ __   ____  __ __  _     ______      ____  
|  |  | /    |  /  ]|  |/ ]    |      ||  |  |  /  _]    |  |  | /    ||  |  || |   |      |    |    |  
|  |  ||  o  | /  / |  ' /     |      ||  |  | /  [_     |  |  ||  o  ||  |  || |   |      |     |  |  
|  _  ||     |/  /  |    \     |_|  |_||  _  ||    _]    |  |  ||     ||  |  || |___|_|  |_|     |  |  
|  |  ||  _  /   \_ |     \      |  |  |  |  ||   [_     |  :  ||  _  ||  :  ||     | |  |       |  |  
|  |  ||  |  \     ||  .  |      |  |  |  |  ||     |     \   / |  |  ||     ||     | |  |       |  |  
|__|__||__|__|\____||__|\_|      |__|  |__|__||_____|      \_/  |__|__| \__,_||_____| |__|      |____|  
  
  
Hey. Detective Kane here. I found a vault, right by the river burried two feet underground.  
I dug it up, seems like I need a password. Do you know what it is?  
Enter the password: th3M0ssM4ni5h3re,y0uc4ntcatchm3  
It worked. The clues he left behind makes me believe that this case is not over just yet. We will need to continue our mission, and stop the Moss Man at all costs.  
bushbash{th1s-is-just-th3-beginning!}
```

`bushbash{th1s-is-just-th3-beginning!}`

___

## xored

**Category**: crypto

**Author**: Harold Gao

**Description**: Oops! I accidentally corrupted my flag with a simple encryption algorithm... I forgot the key though!

`encrypt.py`:
```python
#!/usr/bin/env python3

with open("key", "rb") as keyf:
    key = keyf.read()

if not key:
    raise ValueError("The key file is empty")

with open("flag.txt", "rb") as flagf:
    flag = flagf.read()

encrypted = bytes(
    byte ^ key[i % len(key)]
    for i, byte in enumerate(flag)
)

with open("flag.enc", "wb") as flagencf:
    flagencf.write(encrypted)
```

`flag.enc`:
```shell
cat flag.enc  
XN��{�;pAO��a�:5UI��V�eLU��k�i
```

The solution relies on a known plaintext attack. By XORing the first few bytes of the encrypted flag with the known prefix (`bushbash{`) the key will appear. 

Solve script:
```python
#!/usr/bin/env python3

def xor_bytes(data, key):
    return bytes(data[i] ^ key[i % len(key)] for i in range(len(data)))

def main():
    try:
        with open("flag.enc", "rb") as f:
            ciphertext = f.read()
    except FileNotFoundError:
        print("flag.enc not found")
        return

    known_prefix = [b"bushbash{"]

    print("Starting Known Plaintext Attack...")

    for prefix in known_prefix:
        # extract the key by XORing ciphertext with known plaintext
        derived_key = bytes(c ^ p for c, p in zip(ciphertext[:len(prefix)], prefix))
        
        # get the key length
        for key_len in range(1, len(derived_key) + 1):
            key_guess = derived_key[:key_len]
            
            # verify if this key length matches derived key sequence
            is_valid_length = True
            for i in range(len(derived_key)):
                if derived_key[i] != key_guess[i % key_len]:
                    is_valid_length = False
                    break
            
            if is_valid_length:
                decrypted = xor_bytes(ciphertext, key_guess)
                
                if b"}" in decrypted:
                    print(f"Flag prefix used: {prefix.decode()}")
                    print(f"Recovered Key   : {key_guess}")
                    print(f"Flag            : {decrypted.decode(errors='ignore')}")
                    return

    print("\nfailed :(")

if __name__ == "__main__":
    main()
```

Output:
```shell
python3 solve.py  
Starting Known Plaintext Attack...  
Flag prefix used: bushbash{  
Recovered Key   : b':;\xeb\xb3\x19\x91H\x18'  
Flag            : bushbash{to-x0r-or-nOt-To-Xor}!
```

`bushbash{to-x0r-or-nOt-To-Xor}`
