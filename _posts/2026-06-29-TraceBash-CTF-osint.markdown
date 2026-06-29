---
layout: post
title:  "OSINT Challenges"
date:   2026-06-29 14:14:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /TraceBash-CTF-2026-osint/
---
* TOC
{:toc}

## Echo Chamber

**Category**: osint

**Author:** CYB3RFY

**Description**: A forum user going by **rainyhike22** posted a comment referencing a real-world outdoor activity they recently logged. Analyze the attached forum thread, identify that specific activity, and find its unique public identifying code.

Flag format: `TBCTF{code-in-lowercase}`  
Example flag: `TBCTF{hf12b45}`

I get the challenge files:
```shell
unzip echo_chamber.zip  
Archive:  echo_chamber.zip  
  inflating: player_brief.md  
  inflating: forum_thread.html
```

`player_brief.md`:
```shell
cat player_brief.md  
## Briefing  
We have an ongoing investigation involving a person of interest. As part of our digital forensics, we managed to scrape a single comment thread from a local community forum before it was taken offline.  
  
We suspect the user going by `rainyhike22` is our target. We believe this forum poster's recent outdoor activity can be definitively identified online.  
  
Review the attached forum thread carefully. **Note:** People often vent online and exaggerate or complain about mundane things; not every single detail they mention is necessarily a verifiable digital clue. You need to figure out which part of their post corresponds to a real-world, searchable database.  
  
Find the real-world location/activity they mentioned and submit its identifying code.  
  
**Goal:** Provide the unique identifying code for the activity the user logged.  
**Flag Format:** `TBCTF{<code-lowercase>}` (e.g., `TBCTF{cg12x45}`)  
  
## Assets  
- `forum_thread.html` - A static snapshot of the forum thread.
```

Looking at `forum_thread.html`:

![Alt text](/images/tracebashosint1.png)

I found the geocache they are talking about here: [https://www.geocaching.com/geocache/GCK4GH_son-of-gasworks-scrabble](https://www.geocaching.com/geocache/GCK4GH_son-of-gasworks-scrabble)

`TBCTF{gck4gh}`

___

## Missing Friend

**Category**: osint

**Author:** Vintageitself

**Description**: A tourist went missing after attending a motorsport event in 2025. Two photos were recovered from their phone. Analyze the provided images, identify the bar they visited that night, and find its Google Maps Plus Code.

Flag format: `TBCTF{XXXX+XX}`  
Example flag: `TBCTF{ABCD+EF}`

I get the challenge files:
```shell
unzip missing_friend.zip  
Archive:  missing_friend.zip  
  inflating: MotoGP.jpg  
  inflating: food.jpg
```

`MotoGP.jpg`:

![Alt text](/images/MotoGP.jpg)

`food.jpg`:

![Alt text](/images/food.jpg)

Reverse image searching shows the bar is `Cantina Mexicana Kuta Lombok`, a Mexican restaurant and bar located in Indonesia. I went to the bar on google maps and copied the plus code:
```shell
475G+QQ Kuta, Central Lombok Regency, West Nusa Tenggara, Indonesia
```

`TBCTF{475G+QQ}`
