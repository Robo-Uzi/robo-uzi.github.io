---
layout: post
title:  "Crypto Challenges"
date:   2026-03-30 20:37:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /TexSawCTF-2026-crypto/
---

## `Idiosyncratic Fr*nch`

**Author**: Written by WatTheWat

**Category**: crypto

**Description**: This chall's got a bit of history to it. First, crack this initial cryptogram. Now, apply OSINT tools to find who authors that original script. flag format: `txsaw{first_last}` such as: `txsaw{john_scalzi}`

I get the challenge file:
```shell
cat ciphertext.txt  
Azza wfahv ztu. N rnvy, bndfah na zbfaztv vztak, n vztak ndfa uz n dcnqza zw n uzlvfa, icfuv nmztu. Nthtvutv, rgz gnv gnk n mnk afhgu, vfuv ty mcfadfah nak ytwmcfak. Zg rgnu rnv ugnu rzwk (fv gfv ugzthgu) ugnu wna ugwzthg bp mwnfa ncc afhgu, ugnu fkfzufl rzwk ugnu, gnwk nv F'k uwp uz yta fu kzra, rnv ncrnpv etvu na falg zw urz ztu zi bp hwnvy - izrc zw iztc zw Szr zw Szpnc? - n rzwk rgflg, mp nvvzlfnufza, mwzthgu fauz ycnp na falzahwtztv bnvv nak bnhbn zi aztav, fkfzbv, vczhnav nak vnpfahv, n lzaitvfah, nbzwygztv ztuyzt wfah rgflg F vzthgu fa snfa uz lzauwzc zw utwa zii mtu rgflg rztak nwztak bp bfak n rgfwcrfak zi n lzwk, n rgfycnvg zi n lzwk, n lzwk ugnu rztck vycfu nhnfa nak nhnfa, rztck d  
afu nhnfa nak nhnfa, zi rzwkv rfugztu lzbbtaflnufza zw nap yzvvfmfcfup zi lzbmfanufza, rzwkv rfugztu ywzatalfnufza, vfhafiflnufza zw uwnavlwfyufza mtu ztu zi rgflg, azurfugvun akfah, rnv mwzthgu izwug n ictq, n lzaufatztv, lzbynlu nak ctlfk iczr: na fautfufza, n snlfccnufah iwfvvza zi fcctbfanufza nv fi lnthgu fa n icnvg zi cfhguafah zw fa n bfvu nm wtyucp wfvfah uz tavgwztk na zmsfztv vfha - mtu n vfha, ncnv, ugnu rztck cnvu na favunau zacp uz snafvg izw hzzk.
```

Once I figured out it is a cryptogram I went to [https://www.quipqiup.com/](https://www.quipqiup.com/) and input the ciphertext. I got this passage:
```shell
Noon rings out. A wasp, making an ominous sound, a sound akin to a klaxon or a tocsin, flits about. Augustus, who has had a bad night, sits up blinking and purblind. Oh what was that word (is his thought) that ran through my brain all night, that idiotic word that, hard as I'd try to pun it down, was always just an inch or two out of my grasp - fowl or foul or Vow or Voyal? - a word which, by association, brought into play an incongruous mass and magma of nouns, idioms, slogans and sayings, a confusing, amorphous outpouring which I sought in vain to control or turn off but which wound around my mind a whirlwind of a cord, a whiplash of a cord, a cord that would split again and again, would knit again and again, of words without communication or any possibility of combination, words without pronunciation, signification or transcription but out of which, notwithstanding, was brought forth a flux, a continuous, compact and lucid flow: an intuition, a vacillating frisson of illumination as if caught in a flash of lightning or in a mist abruptly rising to unshroud an obvious sign - but a sign, alas, that would last an instant only to vanish for good.
```

It is from the book La Disparition (1969) by Georges Perec. 

`txsaw{georges_perec}`
