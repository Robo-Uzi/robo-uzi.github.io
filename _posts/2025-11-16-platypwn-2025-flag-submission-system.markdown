---
layout: post
title:  "Flag Submission System"
date:   2025-11-16 22:58:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /platypwn-flag-submission-system/
---

**Title**: Flag Submission System

**Category**: OSINT

**Description**: For this Platypwn, we thought about a new and super secure Flag Submission System! To make it easier for you, we made a tutorial video where you can learn how it works.

I downloaded the challenge file which is `flag-submission-system.mp4`. The video shows someone printing out a form for the "flag submission system". Then they write the first 4 characters of the flag (`PP{n`) on the paper in marker. Then the paper is folded up and sealed in an envelope. 

I use `ffmpeg` to split the video into frames: `ffmpeg -i flag-submission-system.mp4 -vf "fps=30" frame_%04d.png`.

I got 564 images. As I search through, I find 3 frames where the paper is flipped over, and I can see the entire flag. The marker bled through the paper just enough to read the flag. 

Zoomed in screenshot:

![Alt text](/images/mirrored.png)

`PP{n1c3_XR4Y_3y35}`