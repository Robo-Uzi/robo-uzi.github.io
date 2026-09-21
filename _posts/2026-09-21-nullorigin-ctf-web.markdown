---
layout: post
title:  "Web Challenges"
date:   2026-09-21 17:27:00 -0400
author: uzi
tags: [CTF]
permalink: /null-origin-ctf-2026-web/
---
* TOC
{:toc}

## The Forbidden Six

**Category**: web

**By**: CYBxM0nk

**Description**: Three tokens home. One token, one square from victory. The dice reads 6. And the game says no. Someone decided that winning wasn't allowed here. Find out who's actually enforcing that rule — and whether they have the authority to.

I went to the site:

![Alt text](/images/nulloriginweb1.png)

On `https://ludo-the-forbidden-six.onrender.com/app.js` I found api endpoints:
```shell
/api/move
/api/reset
/api/state
```

```shell
curl https://ludo-the-forbidden-six.onrender.com/api/state  
{"ok":true,"state":{"turn":"red","dice":6,"winner":null,"tokens":{"1":{"position":50,"finished":true},"2":{"position":50,"finished":true},"3":{"position":50,"finished":true},"4":{"position":44,"finished":false}}}}

curl -X POST https://ludo-the-forbidden-six.onrender.com/api/reset  
{"ok":true,"state":{"turn":"red","dice":6,"winner":null,"tokens":{"1":{"position":50,"finished":true},"2":{"position":50,"finished":true},"3":{"position":50,"finished":true},"4":{"position":44,"finished":false}}}}

curl -X POST https://ludo-the-forbidden-six.onrender.com/api/move  
{"ok":false,"error":"Invalid token"}
```

On the page I see `Winning moves are disabled by the client.` so I know that I am being blocked client side.

I need to interact with the api directly:
```shell
curl -s -X POST 'https://ludo-the-forbidden-six.onrender.com/api/move' -H 'Content-Type: application/json' --data '{"token":4,"dice":6}' | jq  
{
    "ok": true,
    "winner": "red",
    "state": {
        "turn": "red",
        "dice": 6,
        "winner": "red",
        "tokens": {
            "1": {
                "position": 50,
                "finished": true
            },
            "2": {
                "position": 50,
                "finished": true
            },
            "3": {
                "position": 50,
                "finished": true
            },
            "4": {
                "position": 50,
                "finished": true
            }
        }
    },
    "flag": "NullOrigin{tH3_c1ient_siD3_spe4ks_th3_TrUth}"
}
```

`NullOrigin{tH3_c1ient_siD3_spe4ks_th3_TrUth}`

___

## CTFCeption

**Category**: web

**By**: Cyberhx Team

**Description**: Every archive hides another archive. A clandestine carrier signal transmitting at 25.09 MHz has been intercepted from an encrypted historical vault dedicated to Lokmanya Tilak and preserved by PRX02. Can you investigate the platform, crack the vault, and reconstruct the lost transmission? "Some stories survive because someone keeps telling them."

Going to the site:

![Alt text](/images/nulloriginweb2.png)

I found many endpoints on the page:
```shell
/archive
/archive.js
/archive/evidence/surveillance_report_1897.txt
/archive/evidence/telemetry.json
/archive/old/corrupted_sector_09.bak
/archive/records/kesari_1881_manifesto.txt
/archive/records/swaraj_declaration_1916.txt
/archive/transmissions/kesari_wire_1908.log
/archive/transmissions/signal_intercept.raw
/crypto.js
/downloads
/downloads/extract_vault.py
/downloads/kesari_vault_1908.zip
/evidence/surveillance_report_1897.txt
/evidence/telemetry.json
/extract_vault.py
/images/lokmanya-tilak-archive.png
/images/lokmanya-tilak-collage.png
/kesari_vault_1908.zip
/main.js
/old/corrupted_sector_09.bak
/records/kesari_1881_manifesto.txt
/records/swaraj_declaration_1916.txt
/responsive.css
/style.css
/terminal.css
/terminal.js
/transmissions/kesari_wire_1908.log
/transmissions/signal_intercept.raw
```

I went to `https://ctfception.onrender.com/downloads/extract_vault.py` and I found a python script which creates a zip file with the password `gitarahasya`.

I download a zip file from `https://ctfception.onrender.com/downloads/kesari_vault_1908.zip`.

I unzipped it and got some new files:
```shell
ls  
beacon_payload.enc  MANDALAY_DISPATCH_1908.txt  telegraph_decoder.py

xxd beacon_payload.enc  
00000000: 666e 7037 6448 4e37 6158 6874 636e 4674  fnp7dHN7aXhtcnFt  
00000010: 656e 5276 656e 706e 6633 6437 646e 7838  enRvenpnf3d7dnx8  
00000020: 6251 514a 4351 6f3d                      bQQJCQo=

cat MANDALAY_DISPATCH_1908.txt  
================================================================================  
ARCHIVE INTELLIGENCE REPORT // CONFIDENTIAL TRANSMISSION #1908-MK  
SOURCE: Clandestine Courier Network (Poona - Bombay - Mandalay)  
ARCHIVIST ANNOTATION: PRX02 Deep Memory Chronicles  
STATUS: RESTORED FROM CELLULAR MICRO-RECORD  
================================================================================  
  
HISTORICAL INCIDENT RECORD:  
On 22 July 1908, Lokmanya Bal Gangadhar Tilak was convicted by the British  
Colonial Court under sections 124-A and 153-A of the Indian Penal Code for his  
editorials published in 'Kesari' defending the spirit of freedom and self-rule.  
  
Before sentencing, Tilak addressed the court with words that echoed through generations:  
"In spite of the verdict of the jury, I maintain that I am innocent.  
There are higher powers that rule the destiny of things and it may be the will  
of providence that the cause which I represent may prosper more by my suffering  
than by my remaining free."  
  
Tilak was sentenced to six years of rigorous transportation and banished to  
Mandalay Central Prison in Burma (Myanmar). Cut off from newspapers and political  
allies, he turned his confinement into an eternal fountain of knowledge.  
In his single wooden cell, amidst extreme climate and illness, he penned the  
monumental 400-page manuscript: 'Shrimadh Bhagavad Gita Rahasya' (The Karma Yoga  
interpretation of selfless action).  
  
================================================================================  
FIELD COURIER NOTE (ARCHIVIST: PRX02):  
"Some stories survive because someone keeps telling them. Ideas that refuse  
to disappear cannot be silenced by colonial walls."  
  
Before his secret journals could be intercepted by imperial wardens, an encrypted  
telegraph beacon was generated by the Kesari underground press. The telegraph cipher  
was keyed using the date of the upcoming eternal chronicle:  
Day and Month: 25th Sept -> Key: '2509'  
  
To reconstruct the final archival record:  
1. Decode 'beacon_payload.enc' using telegraph_decoder.py with the chronicle key '2509'.  
2. Enter the resulting beacon phrase into the CTFception Web Terminal using:  
   reconstruct <DECODED_BEACON_KEY>  
================================================================================
```

Running `telegraph_decoder.py` gives me a key:
```
python3 telegraph_decoder.py 2509  
============================================================  
[+] Decoded Telegraph Beacon: LOKMANYA_GATHA_CHRONICLE_1908  
[+] Terminal Command: reconstruct LOKMANYA_GATHA_CHRONICLE_1908  
============================================================
```

Running `reconstruct LOKMANYA_GATHA_CHRONICLE_1908` in the fake terminal on the page gives me the flag:

![Alt text](/images/nulloriginweb3.png)

`NullOrigin{L0km4ny4_G4th4_25S3pt_Sw4r4jy4_PRX02_}`
