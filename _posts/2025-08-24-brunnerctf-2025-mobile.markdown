---
layout: post
title:  "Brunner 2025 Mobile Challenges"
date:   2025-08-24 21:40:10 -0400
author: robo.uzi
tags: [CTF, mobile]
permalink: /brunnerctf-2025-mobile/
---
* TOC
{:toc}

## BakeDown
**Category:** Mobile

**Difficulty:** Beginner  
**Author:** KyootyBella

I just shipped the very first build of my BakeDown app. I haven't done much with it yet, but I did pack it with a few nice _resources_ already. Try loading it onto your phone or an [emulator](https://developer.android.com/studio/run/managing-avds) to try it out!

Oh and btw, did you know an APK is just a ZIP file with assembled Java code and [resources](https://developer.android.com/guide/topics/resources/providing-resources) like strings, fonts, and images? I heard [apktool](https://apktool.org/) might be a good tool for decoding the contents!

I get this file:
```shell
file BakeDown.apk  
BakeDown.apk: Android package (APK), with gradle app-metadata.properties
```

I run `apktool d BakeDown.apk -o BakeDown_apktool` to extract the files. I quickly find the first part of the flag. I look around and can't find the second part. Then I go to [https://www.myandroid.org/](https://www.myandroid.org/) and upload the apk. Once the app runs I can see the first part of the flag again. I go to the next page and I see the second part of the flag.

`brunner{Th1s_Sh0uld_B3_Th3_B3g1nn1ng_0f_Y0ur_M0b1l3_3xp3r13nc3}`

___

## RivalCakes
**Category:** Mobile

**Difficulty:** Easy  
**Author:** KyootyBella

The secret Brunner and Othello cults are waging their battles in the streets of Denmark. One day, you see a member of the Othello cult drop their phone out of their pocket. You grab it, but it seems to be a burner phone. You find only two custom apps, but when opening one, it instantly deletes itself - maybe there is still some data left?

In the cake cults, _preferences_ matter - so keep an eye out and mind what you say. Your Othello rivals are planning a secret meetup, find out where and find the password to infiltrate them!

**Flag format:** The flag is formatted as `brunner{<w3w_code>_<password>}`, where `<w3w_code>` is the [what3words](https://what3words.com/) code of the location, without leading slashes. Make sure your "3 word address language" is set to English. _Example:_ `brunner{candy.magnetic.label_hunter2}`

**Download Link:** [mobile_rivalcakes.7z](https://shared-brunnerctf-2025.nbg1.your-objectstorage.com/files/mobile_rivalcakes.7z)  
**Password:** `VerySecurePasswordForRivalCakes_BrunnerCTF2025`

Extract the files with `7z x mobile_rivalcakes.7z -p"VerySecurePasswordForRivalCakes_BrunnerCTF2025"`

I see 2 directories. `data` and `sdcard`. I see an interesting db file in `data/data/dk.brunnerctf.rivalcakes/databases/orders.db`:
```shell
sqlite3 databases/orders.db
SQLite version 3.40.1 2022-12-28 14:03:47  
Enter ".help" for usage hints.
sqlite> .tables
android_metadata  orders
sqlite> select * from orders;  
16|Alice|Extra frosting|delivered|2025-08-10 14:32:11  
17|Bob|Meet at location in EXIF|pending|2025-08-11 09:21:44  
18|Claire|part1:Othello_is_|cancelled|2025-08-12 08:59:02  
```
This gives part 1 to the password and a hint to look in exif data for the meet location.

```shell
find . -type f -iname '*.jpg' -o -iname '*.png'  
./sdcard/Android/data/com.google.android.gms/files/gmsnet2.jpg  
./sdcard/Download/Cakes/best_cake_in_the_world.jpg  
./sdcard/Download/Cakes/Bored_4_l1fe.jpg  
./sdcard/Download/Cakes/Choco_lovers.jpg  
./sdcard/Download/Cakes/cuppies.jpg

exiftool ./sdcard/Download/Cakes/best_cake_in_the_world.jpg  
User Comment                    : part3:Than_an_ugly_  
GPS Latitude                    : 55 deg 40' 29.56" N  
GPS Longitude                   : 12 deg 33' 52.40" E  
GPS Position                    : 55 deg 40' 29.56" N, 12 deg 33' 52.40" E
```
`https://gps-coordinates.org/my-location.php?lat=55.67487777777777&lng=12.564555555555556`

I put `55.67487, 12.5645` into https://what3words.com/. Get part 2 and 4 of the password:
```shell
grep -r "part2" .  
./data/data/dk.brunnerctf.rivalcakes/shared_prefs/my_prefs.xml:    <string name="step2_data">part2:always_better_</string>

grep -r "part4" .  
./data/data/dk.brunnerctf.rivalcakes/files/menu_temp.html:<html><body><p>part4:Brunsviger</p></body></html>
```
`brunner{grain.slam.harvest_Othello_is_always_better_Than_an_ugly_Brunsviger}`

___
