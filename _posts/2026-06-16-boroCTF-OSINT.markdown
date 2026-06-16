---
layout: post
title:  "OSINT Challenges"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /boroCTF-2026-osint/
---
* TOC
{:toc}

## Boro Hero

**Author**: MK

**Category**: OSINT

**Description**: This famous Freehold High School alum skipped his graduation and became a best-selling artist. Flag Format: `boroCTF{first_last}`

[https://simple.wikipedia.org/w/index.php?oldid=6842305&title=Bruce_Springsteen](https://simple.wikipedia.org/w/index.php?oldid=6842305&title=Bruce_Springsteen)

`boroCTF{Bruce_Springsteen}`

___

## Nature's Takeover

**Author**: ForeverFlames

**Category**: OSINT

**Description**: You can't beat nature. What is this? Example if the flag was "`not the flag`": `boroCTF{not_the_flag}`

challenge image:

![Alt text](/images/nature.png)

I found a report about the ship here: [https://www.theage.com.au/traveller/inspiration/ss-ayrfield-homebush-bay-the-strange-sydney-harbour-shipwreck-that-grew-a-forest-20200923-h1qx1b.html](https://www.theage.com.au/traveller/inspiration/ss-ayrfield-homebush-bay-the-strange-sydney-harbour-shipwreck-that-grew-a-forest-20200923-h1qx1b.html)

`boroCTF{ss_ayrfield}`

___

## The Squad

**Author**: ForeverFlames

**Category**: OSINT

**Description**: What is the name of the team that organized boroCTF?

Its on the ctftime page for the event: [https://ctftime.org/event/3309](https://ctftime.org/event/3309)

`boroCTF{KyteBytes}`

___

## Physical Access >>

**Author**: Solarity

**Category**: OSINT

**Description**: Help!! I forgot my password and got locked out of my Windows PC!

I need to get back in, what can I do?!

What is one of the two files I can exploit in order to get back into my PC?

Flag format: `boroCTF{explorer.exe}`

I found this guide:
- **Boot from external media**  
    Use a Windows installation USB, a Linux live USB, or any recovery environment that gives you command‑line access to the PC’s hard drive.

- **Navigate to the system32 folder**  
    On the locked Windows installation, the critical files live in `C:\Windows\System32` (drive letter may vary).

- **Replace `utilman.exe` with `cmd.exe`**  
    From the recovery command prompt:
```shell
move C:\Windows\System32\utilman.exe C:\Windows\System32\utilman.bak
copy C:\Windows\System32\cmd.exe C:\Windows\System32\utilman.exe
```

- **Reboot normally into the locked Windows login screen**

- **Trigger the backdoor**  
    Click the **Ease of Access** icon (or press `Win + U`). Instead of Utility Manager, you get a **command prompt running as SYSTEM** (the highest privilege level on Windows).

- **Reset the password**  
    From that SYSTEM command prompt, you can change any user’s password:
```cmd
net user YourUsername NewPassword
```

Or create a new admin account:
```cmd
net user hacker Pass123 /add
net localgroup administrators hacker /add
```

- **Log in** with the new credentials.

`boroCTF{utilman.exe}`

___

## Oops...

**Author**: Solarity

**Category**: OSINT

**Description**: My friend (again) was telling me a story about how around two years ago, he may or may not have accidentally pushed a change to a file that disrupted millions of systems at an unprecedented scale.

What was the general name of the configuration file that he modified that broke everything?

Flag Format: `boroCTF{file_name_67}`

They are referring to the global IT outage caused by a faulty configuration file from CrowdStrike. The file known as `Channel File 291` caused a blue screen of death on millions of windows systems. 

`boroCTF{channel_file_291}`

___

## Mansion

**Author**: ForeverFlames

**Category**: OSINT

**Description**: Who made this build?

flag format: `boroCTF{build_maker}`

I get the challenge file:

![Alt text](/images/Mansion.png)

I found the build here: [https://fmcpe.com/maps/883-wayne-manor.html](https://fmcpe.com/maps/883-wayne-manor.html) after reverse image searching with google lens.

`boroCTF{iIRaptorIi}`

___

## Fireman

**Author**: N/A

**Category**: OSINT

**Description**: Here's a real easy OSINT to give you guys a small break.

Find the name of the man in this image. FULL name including middle initial.

Then wrap it with `boroCTF{}` yourself. (Replace spaces with an underscore) Example flag: `boroCTF{forever_f_flames}`

I get the challenge file `fireman.jpg`:

![Alt text](/images/fireman.jpg)

I reverse image search and find the image was originally posted here: [https://www.facebook.com/groups/410267209587130/posts/1847388779208292/](https://www.facebook.com/groups/410267209587130/posts/1847388779208292/)

`boroCTF{Peter_C_Lukens}`

___

## Geopro 1

**Author**: ForeverFlames

**Category**: GEOSINT

**Description**: Find the city this image was taken in.

flag format: `boroCTF{city_name}` (Replace any space with an underscore)

The challenge file is `geosint.png`:

![Alt text](/images/geosint.png)

Reverse image search shows it is an image of the Laxmi Vilas Palace located in Vadodara in the state of Gujarat, India.

`boroCTF{Vadodara}`

___

## Where there's smoke, there's fire.

**Author**: Franklin

**Category**: OSINT

**Description**: My dad told me a story about how around the time of Halo 3's release, when he was driving home, he saw a massive plume of smoke in the distance while he was coming off of Butterfly Court onto Turtle Creek. What's even crazier is that that a Google Maps car was right behind him. I wonder if it ever got onto the maps? To this day, he's wondered if it was captured on camera. Do you know the former name of the street where the fire occured? Flag Format: `boroCTF{Street_Suffix}`

I found the intersection of Butterfly Court and Turtle Creek Drive on google maps in `Gibson, AR 72120, USA`. I went into street view and changed the date to September 2007. In the distance you can see the fire. It was on Eagle Point Drive.

[google maps link](https://www.google.com/maps/place/Butterfly+Ct,+Gibson,+AR+72120,+USA/@34.8869086,-92.2373216,87a,90y,257.52h,100.35t/data=!3m11!1e1!3m9!1svOlYOwkQom9wAkrNUdby2w!2e0!5s20070901T000000!6shttps:%2F%2Fstreetviewpixels-pa.googleapis.com%2Fv1%2Fthumbnail%3Fcb_client%3Dmaps_sv.tactile%26w%3D900%26h%3D600%26pitch%3D-10.351041335721007%26panoid%3DvOlYOwkQom9wAkrNUdby2w%26yaw%3D257.5244293784535!7i3328!8i1664!9m2!1b1!2i33!4m6!3m5!1s0x87d29689db97b555:0x8c8593e53e1e1989!8m2!3d34.8865428!4d-92.2371679!16s%2Fg%2F1hc5wrhk6?entry=ttu&g_ep=EgoyMDI2MDYxMC4wIKXMDSoASAFQAw%3D%3D)

`boroCTF{Eagle_Point_Dr}`

___

## Geopro 3

**Author**: ForeverFlames

**Category**: GEOSINT

**Description**: What is the really tall building I'm looking at... flag format: `boroCTF{building_name}` (Replace any space with an underscore)

I get the challenge image:

![Alt text](/images/geosint3.jpg)

I reverse image search the image and find the hotel is `JW Marriott Edmonton ICE District`. [google maps link](https://www.google.com/maps/place/JW+Marriott+Edmonton+ICE+District/@53.5456526,-113.4955769,669a,90y,306.26h,122.7t/data=!3m10!1e1!3m8!1s6xwwXPhbV5UI9QfXdaKOuw!2e0!6shttps:%2F%2Fstreetviewpixels-pa.googleapis.com%2Fv1%2Fthumbnail%3Fcb_client%3Dmaps_sv.tactile%26w%3D900%26h%3D600%26pitch%3D-32.69866979848668%26panoid%3D6xwwXPhbV5UI9QfXdaKOuw%26yaw%3D306.264992403891!7i16384!8i8192!9m2!1b1!2i50!4m20!1m10!3m9!1s0x53a0233956f98715:0x8e4a13536ee9f84a!2sJW+Marriott+Edmonton+ICE+District!5m2!4m1!1i2!8m2!3d53.54565!4d-113.49582!16s%2Fg%2F11f658mbm8!3m8!1s0x53a0233956f98715:0x8e4a13536ee9f84a!5m2!4m1!1i2!8m2!3d53.54565!4d-113.49582!16s%2Fg%2F11f658mbm8?entry=ttu&g_ep=EgoyMDI2MDYxMC4wIKXMDSoASAFQAw%3D%3D). I looked at the map and saw `Edmonton Tower` was nearby. It was accepted. 

`boroCTF{Edmonton_Tower}`

___

## Geopro 2

**Author**: ForeverFlames

**Category**: GEOSINT

**Description**: What is the name of the nearest restaurant??? I'm hungry!

flag format: `boroCTF{foreverflames_pizzeria}` (Replace any space with an underscore)

I get the challenge image:

![Alt text](/images/geosint2.jpg)

I reverse image search and on google and I found a `booking.com` listing: [https://www.booking.com/hotel/uz/novaia-3-kh-komnatnaia-kvartira-mechta.ru.html?chal_t=1781466698502](https://www.booking.com/hotel/uz/novaia-3-kh-komnatnaia-kvartira-mechta.ru.html?chal_t=1781466698502)

The location is in Bukhara, Uzbekistan. In one of the images on `booking.com` I see a restaurant nearby called `Orzu Burger`.

`boroCTF{Orzu_Burger}`

___

## Hidden Meaning

**Author**: Solarity

**Category**: OSINT

**Description**: I really used to love this one song, but I forgot the name. All I remember is that it had a SUPER dark message and it came out maybe around 10-15 years ago? Now that I think about it, I think it was something about running? Flag format: `boroCTF{song_name}`

[https://en.wikipedia.org/wiki/Pumped_Up_Kicks](https://en.wikipedia.org/wiki/Pumped_Up_Kicks)

`boroCTF{Pumped_Up_Kicks}`

___

## Geopro 4

**Author**: ForeverFlames

**Category**: GEOSINT

**Description**: Find the streetname. :D flag format: `boroCTF{street_name}`

I get the challenge image:

![Alt text](/images/street.png)

I actually didnt reverse image search this one. I just looked up "Pizza Junction" on google maps.

[google maps link](https://www.google.com/maps/place/Pizza+Junction/@11.5829545,122.7516336,9a,75y,199.54h,88.22t/data=!3m7!1e1!3m5!1spFZ9oYC7C4UDEMRHWohd7g!2e0!6shttps:%2F%2Fstreetviewpixels-pa.googleapis.com%2Fv1%2Fthumbnail%3Fcb_client%3Dmaps_sv.tactile%26w%3D900%26h%3D600%26pitch%3D1.775888146946187%26panoid%3DpFZ9oYC7C4UDEMRHWohd7g%26yaw%3D199.54441508366978!7i16384!8i8192!4m14!1m7!3m6!1s0x33a5f3d549fea4c1:0xd38ae71848b61a62!2sPizza+Junction!8m2!3d11.5827496!4d122.7514096!16s%2Fg%2F11ks2mfrwr!3m5!1s0x33a5f3d549fea4c1:0xd38ae71848b61a62!8m2!3d11.5827496!4d122.7514096!16s%2Fg%2F11ks2mfrwr?entry=ttu&g_ep=EgoyMDI2MDYxMC4wIKXMDSoASAFQAw%3D%3D)

The address is: `2068 Rizal Street, Roxas City, Capiz, Philippines`

`boroCTF{Rizal_Street}`

___

## Geopro 5

**Author**: ForeverFlames

**Category**: GEOSINT

**Description**: What country am I in? Note: **ONLY 5 GUESSES** flag format: `boroCTF{country}`

Challenge image:

![Alt text](/images/Country.png)

In the image I see a phone number on a sign: `95967008`

I went to: [https://en.wikipedia.org/wiki/Telephone_numbers_in_Oman](https://en.wikipedia.org/wiki/Telephone_numbers_in_Oman)

`boroCTF{Oman}`

___

## Third Time's the Charm

**Author**: Solarity

**Category**: OSINT

**Description**: My friend recently found vulnerabilities that got him banned off two platforms. What's my friend's handle? Flag Format: `boroCTF{user_name}`

I knew about this one. Ive been seeing news about what this researcher has been dropping: [https://cybernews.com/security/gitlab-bans-rogue-researcher-releasing-windows-zero-days/](https://cybernews.com/security/gitlab-bans-rogue-researcher-releasing-windows-zero-days/)

`boroCTF{Nightmare_Eclipse}`
