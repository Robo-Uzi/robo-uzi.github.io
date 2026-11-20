---
layout: post
title:  "Crypto Challenges"
date:   2026-11-20 01:01:00 -0400
author: uzi
tags: [CTF]
permalink: /spooky-ctf-2026-crypto/
---
* TOC
{:toc}

## Heads will Roll

**Category**: crypto

**Author**: Inferno

**Description**: While chasing Ichabod Crane through Sleepy Hollow, the Headless Horseman got as close as he could, then threw his head! While rolling, rolling, rolling, rolling, through the air, his pumpkin head shouted…what? His perfect aim knocked Ichabod to the ground near the rails, but what did he say?

Challenge file:
```shell
cat pumpkin_head_message.txt  
Etu_taip5_w5n_hpmnw_hlgsdso{lyh0_d}jyi_piihef_ne_oy3p0lh4oskt_i:k3l3
```

It is a rail fence cipher. Decode with key `4`. [cyberchef link](https://gchq.github.io/CyberChef/#recipe=Rail_Fence_Cipher_Decode(4,0)&input=RXR1X3RhaXA1X3c1bl9ocG1ud19obGdzZHNve2x5aDBfZH1qeWlfcGlpaGVmX25lX295M3AwbGg0b3NrdF9pOmszbDM). Output:
```shell
Enjoy_this_pumpkin_with_the_flag_inside:_spooky{5l33py_h0ll0w_h34d5}
```

`spooky{5l33py_h0ll0w_h34d5}`

___

## Numerophobia

**Category**: crypto

**Author**: s3b

**Description**: I went exploring the house of the crazy old man on the corner of the street. He disappeared last weekend and nobody has seen him since. Rumor has it that he was last seen walking out at midnight screaming random numbers in tongues never spoken before. I decided to go and investigate, unsurprisingly the front door was left unlocked. On the way to the stairs I almost tripped over a stack of books, one was about dyslexia I think. For some reason every other step was missing and marked with a 13 on the wall. I finally reached the top and headed towards the light at the end of the hallway. Taped on the wall was a piece of paper…

I get the challenge file:
```shell
cat numerophobia.txt  
23 50 65 50 65 50 65 50 65 50 65 50 65 50 65 50 65 50 65 50 65 50 65 50 65 50 65 50 65 50 65 50 65 50 65 50 65 50 65 10 138 50 65 95 123 51 64 98 108 53 134 52 132 108 65 95 66 52 117 95 127 51 132 53 123 52 108 51 117 116 136 121 120 111 124 112 128 10 63 52 63 52 63 52 63 52 63 52 63 52 63 52 63 52 63 52 63 52 63 52 63 52 63 52 63 52 63 52 63 52 63 52 63 52 63 52
```

Subtract `13` from every other number (0 based even indices, odd ones stay the same), then convert to ASCII and reverse:
```python
python3  
Python 3.13.5 (main, Aug 10 2026, 12:06:59) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> nums = list(map(int, open("numerophobia.txt").read().split()))  
... i = nums.index(10)  
... j = nums.index(10, i + 1)  
... mid = nums[i+1:j]  
... print("".join(chr(n - 13 if k % 2 == 0 else n) for k, n in enumerate(mid))[::-1])  
...  
spooky{th3_4n5w3r_h45_4lw4y5_b33n_42}
```

`spooky{th3_4n5w3r_h45_4lw4y5_b33n_42}`

___

## I Feel Like Someone's Watching Me

**Category**: crypto

**Author**: inferno

**Description**: A letter is found beside a tombstone in a graveyard, but it seems to have been scrambled around.

I get the challenge file:
```shell
cat graveyard_letter.txt
Ypejpof Erpveaypj,

Pmpj odagp zlw rptf, D'mp tprf zlwj eiopagp. Iwf D'mp erol tprf fcp qjpopagp lt ealfcpj, fclwuc D yla'f nalx xcl df do. Al heffpj xcpjp D ul, fcpjp'o fcdo tpprdau fcef tlrrlxo hp. Dt D pmpj fcdan df'o ulap, D'rr upf e owyypa gcdrr wq hz oqdap fl jphday hp. Iwf D apmpj opp fcph. Dt D fwja fl rlln, D'rr larz opp kwdgn treocpo lt cedj lj grlfcpo, iwf fcpz'rr ip ulap iz fcp fdhp D jlway fcp gljapj.

Df ofejfpy xcpa D xeo da gjlxypy ejpeo, rdnp ef fcp herr xp wopy fl ul fl pmpjz xppnpay. Fcp lap xcpjp xp xlwry bwof xern ejlway eay xdaylxoclq tlj clwjo. Zlw nalx df. Lj ef rpeof zlw ydy. Fcpa D ofejfpy tpprdau df ef xljn, iwf fclwucf df xeo bwof fcp gjppqz qlofpjo fcpz ceau pmpjzxcpjp. Fclop fcdauo cemp erxezo tjpenpy hp lwf, zlw nalx fcef. Fcpz hpoo xdfc hz yjpeho pmpa fclwuc D oclwry ip wopy fl fcph iz alx.

Iwf e xppn eul, D ofejfpy tpprdau df ef clhp. Xcpa D xeo gllndau ydaapj tlj bwof hp, xefgcdau Cerrhejn hlmdpo da zlwj lry rdmdau jllh, lj xerndau Rdffrp Erpv lwfodyp. Ocp hdoopo zlw iz fcp xez. Ocp apmpj rdnpy hp hwgc, iwf ocp'o erxezo gewodau e jegnpf alx. Ocp wopy fl ip ol gwfp xcpa zlw xpjp ejlway, pmpa dt ocp xeo erxezo fjzdau fl adq hp.

Df'o ulffpa xljop fcp reof glwqrp aducfo. D'rr tppr olhplap xefgcdau hp xcdrp D'h fjzdau fl orppq, iwf D'h da lwj clwop err erlap xdfc fcp erejh la eay pmpjz yllj eay xdaylx rlgnpy eay glmpjpy. Zlw'y fcdan D'h ol qejealdy dt zlw xpjp ofdrr cpjp. 

D'mp fjdpy fl rlln tlj zlw, iwf df'o rdnp zlw ydoeqqpejpy. Pmpjzxcpjp zlw wopy fl ul fl "upf olhp edj" do bwof hdoodau zlw. Fcpjp'o larz lap qregp D cempa'f fjdpy zpf. D nalx zlw wopy fl glhp mdodf zlwj hlfcpj pmpjz Owayez, iwf D apmpj xeafpy fl lmpjofpq. Iwf D'mp jwa lwf lt lqfdlao. D cemp alfcdau rptf cpjp, ol D'h uldau fl rpemp. Iwf D'rr qwf e grwp fcef oclwry cprq zlw tday hp dt zlw ofdrr gejp.

pzps ejp xefgcdau hp. Bwof jphphipj waypjogljpo daofpey lt oqegpo eay fl xjeq df gljjpgfrz.

D clqp zlw tday hp iptljp fcpz yl.

Odagpjprz,

Fcp Lap Zlw Rptf Ipcday
```

I put the text into [https://www.quipqiup.com/](https://www.quipqiup.com/) (quipqiup is a fast and automated cryptogram solver by [Edwin Olson](http://april.eecs.umich.edu/people/ebolson).). The site decoded it through statistics:
```shell
Dearest Alexander, Ever since you left, I've felt your absence. But I've also felt the presence of another, though I don't know who it is. No matter where I go, there's this feeling that followsme. If I ever think it's gone, I'll get a sudden chill up my spine to remind me. But I never see them. If I turn to look, I'll only see quick flashes of hair or clothes, but they'll be gone by the time I round the corner. It started when I was in crowded areas, like at the mall we used to go to every weekend. The one where we would just walk around and windowshop for hours. You know it. Or at least you did. Then I started feeling it at work, but thought it was just the creepy posters they hang everywhere. Those things have always freaked me out, you know that. They mess with my dreams even though I should be used to them by now. But a week ago, I started feeling it at home. When I was cooking dinner for just me, watching Hallmark movies in your old living room, or walking Little Alex outside. She misses you by the way. She never liked me much, but she's always causing a racket now. She used to be so cute when you were around, even if she was always trying to nip me. It's gotten worse the last couple nights. I'll feel someone watching me while I'm trying to sleep, but I'm in our house all alone with the alarm on and every door and window locked and covered. You'd think I'm so paranoid if you were still here. I've tried to look for you, but it's like you disappeared. Everywhere you used to go to "get some air" is just missing you. There's only one place I haven't tried yet. I know you used to come visit your mother every Sunday, but I never wanted to overstep. But I've run out of options. I have nothing left here, so I'm going to leave. But I'll put a clue that should help you find me if you still care. eyez are watching me. Just remember underscores instead of spaces and to wrap it correctly. I hope you find me before they do. Sincerely, The One You Left Behind
```

`spooky{eyez_are_watching_me}`
