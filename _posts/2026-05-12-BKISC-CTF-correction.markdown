---
layout: post
title:  "Correction"
date:   2026-05-12 19:09:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /BKISC-CTF-2026-correction/
---

**Category**: Misc

**Description**: ^💢😭😭😭 ^💢😭💢 ^😭😭 ^😭😭😭 ^💢😭💢😭 💢💢💢😭💢😭 😭💢💢😭 😭😭😭😭 😭😭😭😭💢 😭😭 😭😭💢💢😭💢 💢😭💢😭 😭😭😭😭 😭💢💢💢💢 💢😭 😭😭😭😭 😭😭💢💢😭💢 💢😭😭 💢💢💢💢💢 💢😭 💢😭💢😭💢💢 💢😭💢😭💢💢 💢😭💢😭💢💢 💢💢💢😭😭💢

I take the description and put it into cyberchef. I do the recipe `find/replace 💢` with `-` and `find/replace 😭` with `.`. I get this morse code:
```shell
^-... ^-.- ^.. ^... ^-.-. ---.-. .--. .... ....- .. ..--.- -.-. .... .---- -. .... ..--.- -.. ----- -. -.-.-- -.-.-- -.-.-- ---..-
```

Then do `from morse` and you get this string:
```shell
PH4I_CH1NH_D0N!!!
```
This is the Vietnamese phrase "Phải chỉnh đốn" which translates to "Must be corrected." Once I put the flag in lowercase it is accepted.

Cyberchef link: [cyberchef](https://gchq.github.io/CyberChef/#recipe=Find_/_Replace(%7B'option':'Regex','string':'%F0%9F%92%A2'%7D,'-',true,false,true,false)Find_/_Replace(%7B'option':'Regex','string':'%F0%9F%98%AD'%7D,'.',true,false,true,false)From_Morse_Code('Space','Line%20feed')&input=XvCfkqLwn5it8J%2BYrfCfmK0gXvCfkqLwn5it8J%2BSoiBe8J%2BYrfCfmK0gXvCfmK3wn5it8J%2BYrSBe8J%2BSovCfmK3wn5Ki8J%2BYrSDwn5Ki8J%2BSovCfkqLwn5it8J%2BSovCfmK0g8J%2BYrfCfkqLwn5Ki8J%2BYrSDwn5it8J%2BYrfCfmK3wn5itIPCfmK3wn5it8J%2BYrfCfmK3wn5KiIPCfmK3wn5itIPCfmK3wn5it8J%2BSovCfkqLwn5it8J%2BSoiDwn5Ki8J%2BYrfCfkqLwn5itIPCfmK3wn5it8J%2BYrfCfmK0g8J%2BYrfCfkqLwn5Ki8J%2BSovCfkqIg8J%2BSovCfmK0g8J%2BYrfCfmK3wn5it8J%2BYrSDwn5it8J%2BYrfCfkqLwn5Ki8J%2BYrfCfkqIg8J%2BSovCfmK3wn5itIPCfkqLwn5Ki8J%2BSovCfkqLwn5KiIPCfkqLwn5itIPCfkqLwn5it8J%2BSovCfmK3wn5Ki8J%2BSoiDwn5Ki8J%2BYrfCfkqLwn5it8J%2BSovCfkqIg8J%2BSovCfmK3wn5Ki8J%2BYrfCfkqLwn5KiIPCfkqLwn5Ki8J%2BSovCfmK3wn5it8J%2BSog&oenc=65001)

You can also get the first part of the flag by removing the `^` characters and translating from morse:
```shell
💢😭😭😭 💢😭💢 😭😭 😭😭😭 💢😭💢😭 > -... -.- .. ... -.-. > BKISC
```

`BKISC{ph4i_ch1nh_d0n!!!}`