---
layout: post
title:  "Patriot 2025 OSINT"
date:   2025-11-24 17:19:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /patriot-ctf-2025-osint/
---
* TOC
{:toc}

## Where's Legally Distinct Waldo One

**Category**: OSINT

**Challenge author**: Sid Kumar

**Description**: Find the building where Waldo is. The flag is the building from which this photograph was taken. The full building name is the flag wrapped in `pctf{...}`. For example: `pctf{Burger_House}`. Seperate spaces with underscores. "Apply eyeballs liberally to the image".

The image for the challenge:

![Alt text](/images/noexif.jpg)

I reverse image searched the photo. After a lot of searching I find it is a picture of `10428 Rivanna River Way` or `David King Hall` on the George Mason University campus. But I need the building where the photo was taken from. After looking at the angles the photo could be taken from on google maps I find it was taken from `Horizon Hall`.

`pctf{Horizon_Hall}`

___

## Where's Legally Distinct Waldo Two

**Category**: OSINT

**Challenge author**: Sid Kumar

**Description**: Find the building where Waldo is. The flag is the building from which this photograph was taken. The full building name is the flag. Seperate spaces with underscores. "Apply eyeballs liberally to the image"

Challenge image:

![Alt text](/images/noexif2.jpg)

This is still on the George Mason University campus. The building on the left is `Rogers Hall.` After looking on google maps I can see the photo was taken from `Thompson Hall` at:                  [google maps link](https://www.google.com/maps/@38.833505,-77.3099345,3a,75y,333.04h,78.87t/data=!3m7!1e1!3m5!1s3Iwgu2eLV69j7SuJN_dzLA!2e0!6shttps:%2F%2Fstreetviewpixels-pa.googleapis.com%2Fv1%2Fthumbnail%3Fcb_client%3Dmaps_sv.tactile%26w%3D900%26h%3D600%26pitch%3D11.134781594424737%26panoid%3D3Iwgu2eLV69j7SuJN_dzLA%26yaw%3D333.03658952471346!7i16384!8i8192?entry=ttu&g_ep=EgoyMDI1MTExNy4wIKXMDSoASAFQAw%3D%3D) 

`pctf{Thompson_Hall}`

___

## Where's Legally Distinct Waldo Three

**Category**: OSINT

**Challenge author**: Sid Kumar

**Description**: Find the building where Waldo is. The flag is the building from which this photograph was taken. The full building name is the flag. Seperate spaces with underscores. The building name should have underscores instead of slashes or spaces. "Apply eyeballs liberally to the image"

Challenge image:

![Alt text](/images/noexif3.jpg)

This is a photo of `Mason Pond`. It was easy to find on google maps. I had to go to [this](https://www.google.com/maps/@38.829139,-77.3095701,3a,29.6y,37.39h,85.33t/data=!3m7!1e1!3m5!1sHW1Ov5hbUhApf8V4AKuvkg!2e0!6shttps:%2F%2Fstreetviewpixels-pa.googleapis.com%2Fv1%2Fthumbnail%3Fcb_client%3Dmaps_sv.tactile%26w%3D900%26h%3D600%26pitch%3D4.672956889826281%26panoid%3DHW1Ov5hbUhApf8V4AKuvkg%26yaw%3D37.39235059791733!7i16384!8i8192?entry=ttu&g_ep=EgoyMDI1MTExNy4wIKXMDSoASAFQAw%3D%3D) google maps link (street view) to see the full name of the building. 

`pctf{Center_for_the_Arts_Concert_Hall}`

___
