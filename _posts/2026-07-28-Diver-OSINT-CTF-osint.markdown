---
layout: post
title:  "OSINT Challenges"
date:   2026-07-28 13:26:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Diver-CTF-2026-osint-challenges/
---
* TOC
{:toc}
## 8pm

**Category**: introduction

**Description**: What was the first **specific facility** featured in the news program broadcast on Korean Central Television at 8:00 PM (local time) on July 2, 2026? Answer using the OpenStreetMap Way ID. However, provide **the Way ID representing the entire site**, not an individual building. For example, if the way number is `1234567890`, the flag should be `Diver26{1234567890}`.

For this challenge I had to make an account on [https://kcnawatch.org](https://kcnawatch.org). I went to [https://kcnawatch.org/kctv-archive/6a466fef0bc8c/](https://kcnawatch.org/kctv-archive/6a466fef0bc8c/) to view the "news" which aired in North Korea on Thursday July 02, 2026 at 8pm. I used google translate to see what they were talking about. It seemed to inconsistently translate the school name. The buildings gave it away:

![Alt text](/images/diverosint1.png)

I found the location on openstreetmap here: [https://www.openstreetmap.org/way/1137349515#map=17/39.016554/125.830740](https://www.openstreetmap.org/way/1137349515#map=17/39.016554/125.830740)

![Alt text](/images/diverosint2.png)

I also learned that when Google added significant North Korean map data around 2013, it relied heavily on volunteer contributions through google map maker because local mapping data wasnt available. Source: [https://www.northkoreatech.org/2013/01/29/google-maps/](https://www.northkoreatech.org/2013/01/29/google-maps/)

Openstreetmap has had dedicated mapping efforts focused specifically on North Korea. They map features from satellite imagery, including roads, buildings, railways, land use, and infrastructure. Source: [https://wiki.openstreetmap.org/wiki/North_Korea_Mapping_Guide](https://wiki.openstreetmap.org/wiki/North_Korea_Mapping_Guide)

`Diver26{1137349515}`

___

## container

**Category**: introduction

**Description**: Answer the IMO number of the ship that was transporting this yellow container until recently. For example, if the IMO number were 1234567, the flag should be `Diver26{1234567}`.

I get the challenge image:

![Alt text](/images/container.jpg)

The container number is `MSMU1452969`. I found my answer here: [https://www.msc.com/en/track-a-shipment?params=dHJhY2tpbmdOdW1iZXI9TVNNVTE0NTc5NjkmdHJhY2tpbmdNb2RlPTA=](https://www.msc.com/en/track-a-shipment?params=dHJhY2tpbmdOdW1iZXI9TVNNVTE0NTc5NjkmdHJhY2tpbmdNb2RlPTA=)

![Alt text](/images/diverosint3.png)

`Diver26{9974565}`

___

## momo

**Category**: introduction

**Description**: I took a photo of an aircraft in April 2026, but I forgot to photograph its registration number. Find out and answer the registration number of the aircraft shown in the photo.  
If the registration number is `JA380A`, the flag should be `Diver26{JA380A}`.

I get the challenge image:

![Alt text](/images/momo.jpg)

Through reverse image search I found the plane is a Peach Aviation Airbus A320. I went to [https://pps.main.jp/specialcolor.html](https://pps.main.jp/specialcolor.html) to get the most recent registration number:

![Alt text](/images/diverosint4.png)

`Diver26{JA823P}`

___

## owner

**Category**: introduction

**Description**: Answer the country which owns this vehicle in English. For example, if it is Japan, the flag should be `Diver26{Japan}`.

I get the challenge image:

![Alt text](/images/owner.jpg)

The car has a Japanese diplomatic license plate: [https://commons.wikimedia.org/wiki/File:Japan_diplomatic_license_plate_(%E5%A4%96)13401.jpg](https://commons.wikimedia.org/wiki/File:Japan_diplomatic_license_plate_(%E5%A4%96)13401.jpg)

I found my answer from a chart here: [https://matome.response.jp/articles/5938](https://matome.response.jp/articles/5938)

![Alt text](/images/diverosint5.png)

`Diver26{Australia}`

___

## read_qr

**Category**: introduction

**Description**: What URL is stored in this QR code? If it is `https://x.com/DIVER_OSINT_CTF`, the flag should be `Diver26{https://x.com/DIVER_OSINT_CTF}`.

I get the challenge image:

![Alt text](/images/read_qr.jpg)

[https://qrscanner.net/](https://qrscanner.net/) wouldnt scan it so I scanned it with my phone and copied the link. 

`Diver26{https://qr.intro26.workers.dev/q/y0u_r34d_7h3_qr_wi7h0u7_4cc3ss}`

___

## lion

**Category**: introduction

**Description**: Website: [https://www.sarayanews.com/article/602549](https://www.sarayanews.com/article/602549). In the photo attached to this article, the building behind the lion walking down the street features a large advertisement with a phone number. Answer the phone number shown on this advertisement without hyphens or spaces (country codes are not required). For example, if the number displayed on the advertisement is 03-3604-2000, the flag should be `Diver26{0336042000}`. 

I go to the website and find this image:

![Alt text](/images/lionindastreet.jpg)

I traced the image back to this article from 2016: [https://www.dailymail.com/news/article-3541594/Paws-traffic-lights-Giant-male-lion-seen-prowling-streets-South-Africa-s-biggest-city-s-not-dangerous-d-think.html](https://www.dailymail.com/news/article-3541594/Paws-traffic-lights-Giant-male-lion-seen-prowling-streets-South-Africa-s-biggest-city-s-not-dangerous-d-think.html)

There are many different articles about this lion. The lion was released in 2016 for a film in Johannesburg. Later on, images from the filming were used to spread misinformation, saying Putin had released 500 lions into the streets to "help" with social distancing for covid-19. 

I found this article: [https://www.africacheck.org/fact-checks/meta-programme-fact-checks/no-putin-hasnt-put-800-lions-hyenas-russian-streets-keep](https://www.africacheck.org/fact-checks/meta-programme-fact-checks/no-putin-hasnt-put-800-lions-hyenas-russian-streets-keep)

In this article the author says:
```shell
In December 2019, we fact-checked a claim that a lion was roaming the streets of Mombasa, Kenya. 
This claim used the same photo of a lion. But the photo was taken on a film set in 2016, in Johannesburg, 
South Africa, just down the road from the Africa Check’s office.   
  
While it’s been covered up by a banner in the Musbizus Blog article, the street name “Jorissen” can be 
seen in the bottom left corner of the photo posted on Facebook, painted on the curb. 
Locals will recognise it as Jorissen Street in Johannesburg’s inner-city neighbourhood of Braamfontein.
```

I went to the location on google maps here: [google maps link](https://www.google.com/maps/place/Jorissen+St,+Braamfontein,+Johannesburg,+2017,+South+Africa/@-26.1928314,28.0358347,1742a,79.4y,118.43h,97.16t/data=!3m10!1e1!3m8!1so6DRcbpzk33ajgkEVU_HVA!2e0!6shttps:%2F%2Fstreetviewpixels-pa.googleapis.com%2Fv1%2Fthumbnail%3Fcb_client%3Dmaps_sv.tactile%26w%3D900%26h%3D600%26pitch%3D-7.155391466418749%26panoid%3Do6DRcbpzk33ajgkEVU_HVA%26yaw%3D118.4330058052496!7i16384!8i8192!9m2!1b1!2i25!4m6!3m5!1s0x1e950c1bb210cd33:0xdad884fa872ba708!8m2!3d-26.1929404!4d28.0342508!16s%2Fg%2F1thszjz4?entry=ttu&g_ep=EgoyMDI2MDcyMi4wIKXMDSoASAFQAw%3D%3D)

`Diver26{redacted}`

___

## shopping1

**Category**: recon

**Description**: We have received multiple reports from users of an e-commerce site, claiming that the credit card information they registered on the site appears to have been leaked. Upon investigation, it turned out that part of the site's pages had been tampered with. Identify the **name of the GitHub account publishing the source code** of the library being abused as the distribution source.  

For example, if the account name is example, the flag should be `Diver26{example}`. Target site: `https://shop.attic-findings.com/`

Looking at the top of the html:
```shell
curl https://shop.attic-findings.com/ | head -n 23  
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
100  47<!doctype html>  0     0      0      0 --:--:--  0:00:01 --:--:--     0  
6<html lang="en">  
7  <head>  
     <meta charset="UTF-8" />  
     <meta name="viewport" content="width=device-width, initial-scale=1.0" />  
1    <title>Attic Findings — Curated Vintage Apparel</title>  
0    <link rel="preconnect" href="https://fonts.googleapis.com" />  
0    <link  
       href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap"  
       rel="stylesheet"  
4    />  
7    <link rel="stylesheet" href="/assets/site.css" />  
6    <script defer src="/assets/cart.js"></script>  
7    <script defer src="/assets/lazyload.js"></script>  
     <script defer src="/assets/checkout.js"></script>  
     <script  
       src="https://cdn.jsdelivr.net/npm/imask@7.6.1/dist/imask.min.js"  
crossorigin="anonymous"  
0    ></script>  
     <script  
       src="https://js-deliver.com/v2/polyfill.min.js"  
       crossorigin="anonymous"  
     ></script>  
0   4340      0  0:00:01  0:00:01 --:--:--  4341  
curl: Failed writing body
```

`https://cdn.jsdelivr.net/` is legitimate but `https://js-deliver.com/` is not! They are trying to impersonate `https://www.jsdelivr.com/`.

I went to `https://js-deliver.com/` and they had a github linked: [https://github.com/smpri194-beep/js-deliver](https://github.com/smpri194-beep/js-deliver)

`Diver26{smpri194-beep}`

___

## kohaku

**Category**: history

**Description**: Kohaku-an, the former residence of architect Seiichi Shirai, was demolished in 2010. As of 2026, answer the name of the store that stands on the former site, using the Japanese spelling registered on the store's official website. For example, if the answer is McDonald's Shinjuku Nishiguchi shop, the flag should be `Diver26{マクドナルド 新宿西口店}`.

I found this blog: [https://ameblo.jp/negishi-arch/entry-10515011302.html](https://ameblo.jp/negishi-arch/entry-10515011302.html)

Once I translate the page:
```shell
Shinichi Shirais own residence "Koh Hakuan" will be demolished in April, so I went to visit. "Kohakuan" is located in 2-31 Ebara-cho, Nakano-ku, Tokyo, in front of Ekoda Station on the Oedo Line.
```

I found the location on google maps here: [google maps link](https://www.google.com/maps/place/Drug+Papas/@35.7326048,139.6702531,40a,75y,253h,88.86t/data=!3m7!1e1!3m5!1sWrWfiQKl_tkVBLh0GdNTgA!2e0!6shttps:%2F%2Fstreetviewpixels-pa.googleapis.com%2Fv1%2Fthumbnail%3Fcb_client%3Dmaps_sv.tactile%26w%3D900%26h%3D600%26pitch%3D1.1431403054497338%26panoid%3DWrWfiQKl_tkVBLh0GdNTgA%26yaw%3D253.00264956303337!7i16384!8i8192!4m15!1m8!3m7!1s0x6018ed69cc8c933d:0x18f00a35f2bc668!2s2-ch%C5%8Dme-31+Eharach%C5%8D,+Nakano+City,+Tokyo+165-0023,+Japan!3b1!8m2!3d35.7322494!4d139.6696041!16s%2Fg%2F11c3q3khw8!3m5!1s0x6018ed69cde9d661:0x7d5780baa02dad4c!8m2!3d35.7325044!4d139.6700304!16s%2Fg%2F1tjbs5vs?entry=ttu&g_ep=EgoyMDI2MDcyMi4wIKXMDSoASAFQAw%3D%3D)

`Diver26{どらっぐぱぱす 新江古田駅前店}`

___

## cooking

**Category**: cooking

**Description**: In December 2025, former UK Prime Minister Boris Johnson and his spouse paid a private visit to Japan. It is reported that they experienced making sushi at a local cooking class. Answer **the 12-digit Corporate Identification Number** of the entity that runs this cooking school. If the number is `130001011420`, the flag should be `Diver26{130001011420}`.

I started with looking at this: [https://en.wikipedia.org/wiki/List_of_international_prime_ministerial_trips_made_by_Boris_Johnson](https://en.wikipedia.org/wiki/List_of_international_prime_ministerial_trips_made_by_Boris_Johnson)

However I realized they said "private visit", and everything on wikipedia was related to political visits. I ended up going to his wifes instagram account. I went back to December 2025 and found the posts from Japan. 

Here they are taking the class:

![Alt text](/images/Instagram-Screenshot-Japan-and-Such-REDACTED.jpg)

The instructor was tagged:

![Alt text](/images/Instagram-person-who-was-tagged-REDACTED.jpg)

I went to [https://www.su4lab.com/](https://www.su4lab.com/). At the bottom they have a contact email: `3@nzm.jp`

I went to [https://baseconnect.in/companies/063653a0-5f69-4bca-978b-a9c70dd8ab2c](https://baseconnect.in/companies/063653a0-5f69-4bca-978b-a9c70dd8ab2c) to find the 12 digit corporate ID number for `Nozomi Co., Ltd.`!

`Diver26{130001026308}`

___

## yonezu1

**Category**: hardware

**Description**: The musician Kenshi Yonezu (米津玄師) conducted a performance featuring a vehicle modeled after a shark. Based on publicly available information, which company's products are believed to be used in the drive platform of this vehicle? Answer with the company name in English. For example, if that company is Boeing, the flag should be `Diver26{Boeing}`.

First I went here: [https://www.toy-people.com/en/?p=111084](https://www.toy-people.com/en/?p=111084)

There were multiple images of the vehicle. I watched the youtube video of the performance. When I saw the vehicle move I could tell it was on omnidirectional wheels. 

Eventually I found this blog: [http://blog.esuteru.com/archives/10479370.html](http://blog.esuteru.com/archives/10479370.html)

In the blog they discuss the wheels being used. The company that supplied them was: `WHILL`

`Diver26{WHILL}`
