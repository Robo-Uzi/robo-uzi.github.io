---
layout: post
title:  "Brunner 2025 OSINT Challenges"
date:   2025-08-24 20:20:30 -0400
author: robo.uzi
tags: [CTF, OSINT]
permalink: /brunnerctf-2025-osint/
---
* TOC
{:toc}

## There Is a Lovely Land
**Category:** OSINT

**Difficulty:** Easy  
**Author:** Toxicd

The Danish national anthem is called "Der er et yndigt land" ("There Is a Lovely Land"), and I fully agree. A friend of mine took this picture, but I don't know the name of the bridge! Please help me find it so I can go see it!

**Flag format:** `brunner{<bridge name in Danish in lowercase>}`

I am given an image of a bridge called `SomeBridge.JPG`.

First I tried reverse image search. I found it at `https://pl.wikipedia.org/wiki/Lista_most%C3%B3w_w_Danii` which listed out Danish bridges. Flag: `brunner{sallingsundbroen}`.

___

## Train Mania
**Category:** OSINT

**Difficulty:** Easy  
**Author:** Quack

I recently stumpled upon this cool train! But I'd like to know a bit more about it... Can you please tell me the operating company, model number and its maximum service speed (km/h in regular traffic)?

The flag format is `brunner{OPERATOR-MODELNUMBER-SERVICESPEED}`. So if the operator you have found is `DSB`, the model number `RB1`, and the maximum service speed is `173` km/h, the flag would be `brunner{DSB-RB1-173}`.

I am given a video: 
```shell
file TrainMania.mp4  
TrainMania.mp4: ISO Media, MP4 Base Media v1 [ISO 14496-12:2003]
```

I start by taking a screenshot of the train in the video at a moment where I can see defining  features (the doors passing by). There was also a small decal on the train. Through reverse image search I find the train model is `X2000`. It was also easy to find the company that owns the trains (Swedish Railways) which gets shorted to `SJ`. I google for its maximum service speed. The flag is: `brunner{SJ-X2000-200}`

___

## Traditional Cake
**Category:** OSINT

**Difficulty:** Medium  
**Author:** OddNorseman

I tried this amazing traditional cake at this amazing place I visited... but now I don't remember what it was called and where I was. All I have is this picture I took some time before. Can you help me find the exact spot I took the image from, so I can try to retrace my steps?

**Flag format:** Use [what3words](https://what3words.com/) to find the 3-word code for the location (without leading slashes) and wrap in `brunner{}`. So, if the location found has code `apricot.surfer.uses`, the flag would be `brunner{apricot.surfer.uses}`. _Note:_ Make sure "3 word address language" is set to English in your what3words settings to get the right code.

I get an image of an old looking building taken from across the water.

I zoom in on the building, take a screenshot, and reverse image search with this. I see one building that looks very similar and search for `Badenburg Castle`. With google maps I confirm it is a picture of `Schloß Nymphenburg 205, 80638 München, Germany`. I use street view to find the place the photo was taken from. I can see the same pov, just taken from a bit further back.

The photo was taken across the water at `Badenburger See, 80638 München, Germany`. I go to `https://what3words.com/` and click around this area until I find the right code. The unique 3 letter sequence identifies a 3 meter by 3 meter area. Eventually I find the flag at: `brunner{rice.underway.pump}`

___

