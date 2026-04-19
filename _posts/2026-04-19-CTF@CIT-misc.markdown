---
layout: post
title:  "Miscellaneous Challenges"
date:   2026-04-19 17:33:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /CTF@CIT-2026-misc/
---
* TOC
{:toc}

## SAM, I am

**Author**: elemental

**Category**: misc

**Description**: I dumped the SAM hive and found a document stating the password policy is 5 characters + complexity. `97a3e51e5a006eccac91e0ceabd4965b`. FLAG FORMAT: `CIT{password}`

I get the NTLM hash `97a3e51e5a006eccac91e0ceabd4965b`. I take it to [https://crackstation.net/](https://crackstation.net/) and get the password `C1t!!`.

`CIT{C1t!!}`

___

## Very Expansive

**Author**: bootstrap

**Category**: misc

**Description**: Where in the wide, wide world of sports is this, beratna? Good place for Dusters, mi pensa. FLAG FORMAT: `CIT{Name_of_Place}`. Example: `CIT{Grand_Canyon}`.

`Good place for Dusters`: In `The Expanse`, Dusters is a nickname for people from Mars. Mariner Valley is one of the largest and most developed settlements on the planet. [https://expanse.fandom.com/wiki/Martian](https://expanse.fandom.com/wiki/Martian). 

``CIT{Mariner_Valley}``

___

## Robots

**Author**: bootstrap

**Category**: misc

**Description**: Beep Boop

```shell
curl https://ctf.cyber-cit.club/robots.txt | tail  
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
100  1047  100  1047    0     0   1030      0  0:00:01  0:00:01 --:--:--  1030  
  
  
  
  
  
  
  
  
  
CIT{S8kMc789Gd37Py1gQPiWbeqxx}
```

`CIT{S8kMc789Gd37Py1gQPiWbeqxx}`

___

## Call me, maybe? No… wrong decade.

**Author**: elemental

**Category**: misc

**Description**: I don't have a witty description for this one... `$2b$10$Ni0U3D5ibg1NY6G/k8CDHuXG7m/WNZzuV/9PDPnRzgKs4wUjaTwGO`. FLAG FORMAT: `CIT{password}`

I get a bcrypt hash: `$2b$10$Ni0U3D5ibg1NY6G/k8CDHuXG7m/WNZzuV/9PDPnRzgKs4wUjaTwGO`. It cracks in hashcat mode `30600`.

The hint is in the title:
```shell
Call me, maybe?          |      Carly Rae Jepsen (Call Me Maybe 2012)
No... wrong decade       |      1980s
```

I asked AI to generate a small custom wordlist and use john because I don't have a GPU in my VM:
```shell
/usr/sbin/john --wordlist=cit_ctf_wordlist.txt --format=bcrypt hash.txt  
Using default input encoding: UTF-8  
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])  
Cost 1 (iteration count) is 1024 for all loaded hashes  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
8675309jenny     (?)  
1g 0:00:00:00 DONE (2026-04-18 00:09) 1.492g/s 53.73p/s 53.73c/s 53.73C/s Elemental..Jenny8675309  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```

`CIT{8675309jenny}`

___

## Dog barking

**Author**: ronnie

**Category**: misc

**Description**: I recorded this audio at the dogpark the other day and I think the dogs were trying to tell me something??? FLAG FORMAT: `CIT{example_flag}`.

I get the challenge file:
```shell
file challenge.wav  
challenge.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 16000 Hz
```

When playing the file I can hear a small dog barking and a big dog barking (2 different synthesised dog barks, there are actually 3). The barks look like this in `audacity`:

![Alt text](/images/barks.png)

By checking each barks amplitude I find:
- 492 Hz (loud / short): bit 0 (A)
- 527 Hz (loud / short): byte separator (B)`
- 590 Hz (quiet / long): bit 1 (C)`

The 527 Hz barks split the stream into 30 groups of exactly 8 barks each. Reading each group as an 8 bit binary number (`A=0`, `C=1`) gives ASCII text.

Solve script:
```python
import wave
import numpy as np

def decode(wav_path):
    # load audio
    with wave.open(wav_path, 'r') as f:
        rate = f.getframerate()
        frames = f.readframes(f.getnframes())
        samples = np.frombuffer(frames, dtype=np.int16).astype(np.float32)

    # segment using energy (50ms windows threshold 10% of max)
    window = int(rate * 0.05)
    energies = []
    for i in range(0, len(samples) - window, window):
        chunk = samples[i:i+window]
        energies.append(np.sqrt(np.mean(chunk**2)))
    energies = np.array(energies)
    threshold = energies.max() * 0.1

    # find contiguous regions above threshold
    in_bark = False
    barks = []  # each: (start_idx, end_idx) in window indices
    start = 0
    for i, e in enumerate(energies):
        if e > threshold and not in_bark:
            in_bark = True
            start = i
        elif e <= threshold and in_bark:
            in_bark = False
            barks.append((start, i))

    # convert window indices to sample indices
    bark_samples = [(s * window, e * window) for s, e in barks]

    # for each bark find dominant frequency (FFT)
    def dominant_freq(start_sample, end_sample):
        chunk = samples[start_sample:end_sample]
        if len(chunk) < 10:
            return 0
        N = 4096
        buf = np.zeros(N)
        buf[:min(len(chunk), N)] = chunk[:min(len(chunk), N)]
        fft_vals = np.abs(np.fft.rfft(buf))
        freqs = np.fft.rfftfreq(N, 1.0/rate)
        # exclude DC
        peak_idx = np.argmax(fft_vals[1:]) + 1
        return freqs[peak_idx]

    freqs = []
    for s, e in bark_samples:
        f = dominant_freq(s, e)
        freqs.append(f)

    # classify: 
    # 492 Hz > A (0) 
    # 527 Hz > B (separator)
    # 590 Hz > C (1)
    def classify(f):
        if abs(f - 527) < 10:
            return 'B'
        elif abs(f - 492) < 15:
            return 'A'
        else:
            return 'C'

    seq = ''.join(classify(f) for f in freqs)

    # split on B and decode groups of 8
    flag_chars = []
    for group in seq.split('B'):
        if len(group) == 8:
            bits = group.replace('A', '0').replace('C', '1')
            flag_chars.append(chr(int(bits, 2)))
            print(bits)
    return ''.join(flag_chars)

if __name__ == '__main__':
    print(decode('challenge.wav'))
```

Output:
```shell
python3 solve3.py  
01000011  
01001001  
01010100  
01111011  
01100010  
00110100  
01110010  
01101011  
01101001  
01101110  
01100111  
01011111  
01110101  
01110000  
01011111  
01110100  
01101000  
00110011  
01011111  
01110111  
01110010  
00110000  
01101110  
01100111  
01011111  
01110100  
01110010  
00110011  
00110011  
01111101  
CIT{b4rking_up_th3_wr0ng_tr33}
```

`CIT{b4rking_up_th3_wr0ng_tr33}`

___

## What's the word?

**Author**: bootstrap

**Category**: misc

**Description**: B-b-b-bird, bird, bird! The flag is in the file, but where?? SHA1: `b2b621068c3632d102c62be9b3cc386c6c175eff`.

I get the challenge file:
```shell
sha1sum file  
b2b621068c3632d102c62be9b3cc386c6c175eff  file  

file file  
file: CDFV2 Encrypted
```

It is a password protected `.docx` file. I get the hash of the file with `office2john`:
```shell
office2john file > officehash

cat officehash  
$office$*2013*100000*256*16*42c71bac48d39fb13c1528f9c39e7b17*4aee5a0a38f0e72f05fd413a96f9b03a*f377ff864c015f83ad20b8bae2cfaceaa24e196932ecde0813adae4306fc131d
```

Crack the password:
```shell
hashcat -m 9600 officehash /usr/share/wordlists/rockyou.txt
... etc
$office$*2013*100000*256*16*42c71bac48d39fb13c1528f9c39e7b17*4aee5a0a38f0e72f05fd413a96f9b03a*f377ff864c015f83ad20b8bae2cfaceaa24e196932ecde0813adae4306fc131d:q1w2e3r4t5  
  
Session..........: hashcat  
Status...........: Cracked  
Hash.Mode........: 9600 (MS Office 2013)  
Hash.Target......: $office$*2013*100000*256*16*42c71bac48d39fb13c1528f...fc131d  
Time.Started.....: Sat Apr 18 02:40:11 2026 (2 mins, 13 secs)  
Time.Estimated...: Sat Apr 18 02:42:24 2026 (0 secs)  
Kernel.Feature...: Pure Kernel  
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)  
Guess.Queue......: 1/1 (100.00%)  
Speed.#1.........:       96 H/s (25.77ms) @ Accel:384 Loops:512 Thr:1 Vec:4  
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)  
Progress.........: 12672/14344386 (0.09%)  
Rejected.........: 0/12672 (0.00%)  
Restore.Point....: 12288/14344386 (0.09%)  
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1  
Candidate.Engine.: Device Generator  
Candidates.#1....: hawkeye -> legion  
Hardware.Mon.#1..: Util: 90%  
  
Started: Sat Apr 18 02:39:31 2026  
Stopped: Sat Apr 18 02:42:27 2026
```

The password is `q1w2e3r4t5`. I open the file with `libreoffice` and get the flag:

![Alt text](/images/encdocx.png)

`CIT{b1rd_1s_th3_w0rd}`
