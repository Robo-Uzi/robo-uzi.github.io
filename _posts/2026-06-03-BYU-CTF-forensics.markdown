---
layout: post
title:  "Forensics Challenges"
date:   2026-06-03 05:46:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /BYU-CTF-2026-forensics/
---
* TOC
{:toc}

## There Will Be Cake

**Category**: Forensics

**Author**: JC6143

**Description**: The Enrichment Center is required to remind you that all test subject activity will be logged, analyzed, and stored for scientific purposes.

"Cake and grief counseling will be available at the conclusion of the test."

*The pcap file contains a flag for each of the following challenges: "There will be cake", "Are you still there?", "Alright. Paradox time", and "Corrupted Cores".  
If the flag you found doesn't work, then it most likely belongs to one of the other 3 challenges.

Hint: what is a baked treat similar to a cake that you can find on almost any website?

I get the challenge file:
```shell
file GLaDOS_Network.pcapng  
GLaDOS_Network.pcapng: pcapng capture file - version 1.0
```

I open the pcap in wireshark and follow http stream 0 and find a base64 encoded cookie:
```http
POST / HTTP/1.1
Host: 192.168.132.133
Cookie: cake=Ynl1Y3Rme1RoM19DNGszXyFzXzRfTCEzX0hUQzU2emVFfQ==
X-Seq: 0
Content-Type: application/json
Content-Length: 24
Connection: close

{"GetChamberNumber": 17}
HTTP/1.1 200 OK
date: Thu, 19 Mar 2026 23:41:42 GMT
server: uvicorn
content-length: 52
content-type: application/json
Connection: close

"{\"TestingChamber\": 17, \"Status\": \"Complete\"}"
```

```shell
echo 'Ynl1Y3Rme1RoM19DNGszXyFzXzRfTCEzX0hUQzU2emVFfQ==' | base64 -d  
byuctf{Th3_C4k3_!s_4_L!3_HTC56zeE}
```

`byuctf{Th3_C4k3_!s_4_L!3_HTC56zeE}`

___

## Are You Still There?

**Category**: Forensics

**Author**: JC6143

**Description**: Forms FORM-55551-6: Personnel File Addendum Addendum:

One last thing:

Go ahead and leave me. I think I prefer to stay inside. Maybe you'll find someone else to help you. Maybe Black Mesa... THAT WAS A JOKE. HA HA. FAT CHANCE. Anyway, this cake is great. It's so delicious and moist. Look at me still talking when there's Science to do. When I look out there, it makes me GLaD I'm not you. I've experiments to run. There is research to be done. On the people who are still alive.

PS: And believe me I am still alive. PPS: I'm doing Science and I'm still alive. PPPS: I feel FANTASTIC and I'm still alive.

FINAL THOUGHT: While you're dying I'll be still alive.

FINAL THOUGHT PS: And when you're dead I will be still alive.

STILL ALIVE

Still alive.

*The pcap file contains a flag for each of the following challenges: "There will be cake", "Are you still there?", "Alright. Paradox time", and "Corrupted Cores".  
If the flag you found doesn't work, then it most likely belongs to one of the other 3 challenges.

Hint: how would you remotely check if a server is online?

I pulled out all `icmp` packets with `tshark`:
```shell
tshark -r GLaDOS_Network.pcapng -Y "icmp" -x  
0000  00 0c 29 b7 fb 48 00 50 56 c0 00 08 08 00 45 00   ..)..H.PV.....E.  
0010  00 20 00 01 00 00 40 01 70 0f 59 6e 6c 31 c0 a8   . ....@.p.Ynl1..  
0020  84 85 08 00 20 23 00 00 00 00 62 79 75 63         .... #....byuc  
  
0000  00 50 56 fc 4f b3 00 0c 29 b7 fb 48 08 00 45 00   .PV.O...)..H..E.  
0010  00 20 c8 7f 00 00 40 01 a7 90 c0 a8 84 85 59 6e   . ....@.......Yn  
0020  6c 31 00 00 28 23 00 00 00 00 62 79 75 63 00 00   l1..(#....byuc..  
0030  00 00 00 00 00 00 00 00 00 00 00 00               ............  
  
0000  00 0c 29 b7 fb 48 00 50 56 c0 00 08 08 00 45 00   ..)..H.PV.....E.  
0010  00 20 00 01 00 00 40 01 8a 0e 59 33 52 6d c0 a8   . ....@...Y3Rm..  
0020  84 85 08 00 08 44 00 00 00 01 74 66 7b 54         .....D....tf{T  
  
0000  00 50 56 fc 4f b3 00 0c 29 b7 fb 48 08 00 45 00   .PV.O...)..H..E.  
0010  00 20 21 6c 00 00 40 01 68 a3 c0 a8 84 85 59 33   . !l..@.h.....Y3  
0020  52 6d 00 00 10 44 00 00 00 01 74 66 7b 54 00 00   Rm...D....tf{T..  
0030  00 00 00 00 00 00 00 00 00 00 00 00               ............  
  
0000  00 0c 29 b7 fb 48 00 50 56 c0 00 08 08 00 45 00   ..)..H.PV.....E.  
0010  00 20 00 01 00 00 40 01 7e 0e 65 31 52 6f c0 a8   . ....@.~.e1Ro..  
0020  84 85 08 00 10 58 00 00 00 02 75 72 72 33         .....X....urr3  
  
0000  00 50 56 fc 4f b3 00 0c 29 b7 fb 48 08 00 45 00   .PV.O...)..H..E.  
0010  00 20 ea 34 00 00 40 01 93 da c0 a8 84 85 65 31   . .4..@.......e1  
0020  52 6f 00 00 18 58 00 00 00 02 75 72 72 33 00 00   Ro...X....urr3..  
0030  00 00 00 00 00 00 00 00 00 00 00 00               ............  
  
0000  00 0c 29 b7 fb 48 00 50 56 c0 00 08 08 00 45 00   ..)..H.PV.....E.  
0010  00 20 00 01 00 00 40 01 af 2c 4d 31 39 51 c0 a8   . ....@..,M19Q..  
0020  84 85 08 00 31 6a 00 00 00 03 74 5f 52 33         ....1j....t_R3  
  
0000  00 50 56 fc 4f b3 00 0c 29 b7 fb 48 08 00 45 00   .PV.O...)..H..E.  
0010  00 20 26 73 00 00 40 01 88 ba c0 a8 84 85 4d 31   . &s..@.......M1  
0020  39 51 00 00 39 6a 00 00 00 03 74 5f 52 33 00 00   9Q..9j....t_R3..  
0030  00 00 00 00 00 00 00 00 00 00 00 00               ............  
  
0000  00 0c 29 b7 fb 48 00 50 56 c0 00 08 08 00 45 00   ..)..H.PV.....E.  
0010  00 20 00 01 00 00 40 01 9d 36 4e 48 4a 30 c0 a8   . ....@..6NHJ0..  
0020  84 85 08 00 26 58 00 00 00 04 64 33 6d 70         ....&X....d3mp  
  
0000  00 50 56 fc 4f b3 00 0c 29 b7 fb 48 08 00 45 00   .PV.O...)..H..E.  
0010  00 20 66 25 00 00 40 01 37 12 c0 a8 84 85 4e 48   . f%..@.7.....NH  
0020  4a 30 00 00 2e 58 00 00 00 04 64 33 6d 70 00 00   J0...X....d3mp..  
0030  00 00 00 00 00 00 00 00 00 00 00 00               ............  
  
0000  00 0c 29 b7 fb 48 00 50 56 c0 00 08 08 00 45 00   ..)..H.PV.....E.  
0010  00 20 00 01 00 00 40 01 79 0e 58 31 64 6f c0 a8   . ....@.y.X1do..  
0020  84 85 08 00 53 6b 00 00 00 05 74 21 30 6e         ....Sk....t!0n  
  
0000  00 50 56 fc 4f b3 00 0c 29 b7 fb 48 08 00 45 00   .PV.O...)..H..E.  
0010  00 20 f7 79 00 00 40 01 81 95 c0 a8 84 85 58 31   . .y..@.......X1  
0020  64 6f 00 00 5b 6b 00 00 00 05 74 21 30 6e 00 00   do..[k....t!0n..  
0030  00 00 00 00 00 00 00 00 00 00 00 00               ............  
  
0000  00 0c 29 b7 fb 48 00 50 56 c0 00 08 08 00 45 00   ..)..H.PV.....E.  
0010  00 20 00 01 00 00 40 01 9f 01 4d 33 49 7a c0 a8   . ....@...M3Iz..  
0020  84 85 08 00 77 3f 00 00 00 06 5f 4c 21 6e         ....w?...._L!n  
  
0000  00 50 56 fc 4f b3 00 0c 29 b7 fb 48 08 00 45 00   .PV.O...)..H..E.  
0010  00 20 35 c3 00 00 40 01 69 3f c0 a8 84 85 4d 33   . 5...@.i?....M3  
0020  49 7a 00 00 7f 3f 00 00 00 06 5f 4c 21 6e 00 00   Iz...?...._L!n..  
0030  00 00 00 00 00 00 00 00 00 00 00 00               ............  
  
0000  00 0c 29 b7 fb 48 00 50 56 c0 00 08 08 00 45 00   ..)..H.PV.....E.  
0010  00 20 00 01 00 00 40 01 76 04 58 30 67 7a c0 a8   . ....@.v.X0gz..  
0020  84 85 08 00 65 51 00 00 00 07 33 73 5f 34         ....eQ....3s_4  
  
0000  00 50 56 fc 4f b3 00 0c 29 b7 fb 48 08 00 45 00   .PV.O...)..H..E.  
0010  00 20 72 22 00 00 40 01 03 e3 c0 a8 84 85 58 30   . r"..@.......X0  
0020  67 7a 00 00 6d 51 00 00 00 07 33 73 5f 34 00 00   gz..mQ....3s_4..  
0030  00 00 00 00 00 00 00 00 00 00 00 00               ............  
  
0000  00 0c 29 b7 fb 48 00 50 56 c0 00 08 08 00 45 00   ..)..H.PV.....E.  
0010  00 20 00 01 00 00 40 01 6a 16 58 30 73 68 c0 a8   . ....@.j.X0sh..  
0020  84 85 08 00 26 76 00 00 00 08 72 33 5f 4e         ....&v....r3_N  
  
0000  00 50 56 fc 4f b3 00 0c 29 b7 fb 48 08 00 45 00   .PV.O...)..H..E.  
0010  00 20 fe 67 00 00 40 01 6b af c0 a8 84 85 58 30   . .g..@.k.....X0  
0020  73 68 00 00 2e 76 00 00 00 08 72 33 5f 4e 00 00   sh...v....r3_N..  
0030  00 00 00 00 00 00 00 00 00 00 00 00               ............  
  
0000  00 0c 29 b7 fb 48 00 50 56 c0 00 08 08 00 45 00   ..)..H.PV.....E.  
0010  00 20 00 01 00 00 40 01 5a ed 62 47 78 7a c0 a8   . ....@.Z.bGxz..  
0020  84 85 08 00 68 30 00 00 00 09 30 74 5f 52         ....h0....0t_R  
  
0000  00 50 56 fc 4f b3 00 0c 29 b7 fb 48 08 00 45 00   .PV.O...)..H..E.  
0010  00 20 ec ec 00 00 40 01 6e 01 c0 a8 84 85 62 47   . ....@.n.....bG  
0020  78 7a 00 00 70 30 00 00 00 09 30 74 5f 52 00 00   xz..p0....0t_R..  
0030  00 00 00 00 00 00 00 00 00 00 00 00               ............  
  
0000  00 0c 29 b7 fb 48 00 50 56 c0 00 08 08 00 45 00   ..)..H.PV.....E.  
0010  00 20 00 01 00 00 40 01 72 06 58 31 6b 77 c0 a8   . ....@.r.X1kw..  
0020  84 85 08 00 a3 1e 00 00 00 0a 21 64 33 73         ..........!d3s  
  
0000  00 50 56 fc 4f b3 00 0c 29 b7 fb 48 08 00 45 00   .PV.O...)..H..E.  
0010  00 20 6c 2a 00 00 40 01 05 dd c0 a8 84 85 58 31   . l*..@.......X1  
0020  6b 77 00 00 ab 1e 00 00 00 0a 21 64 33 73 00 00   kw........!d3s..  
0030  00 00 00 00 00 00 00 00 00 00 00 00               ............  
  
0000  00 0c 29 b7 fb 48 00 50 56 c0 00 08 08 00 45 00   ..)..H.PV.....E.  
0010  00 1d 00 01 00 00 40 01 a1 18 64 58 30 41 c0 a8   ......@...dX0A..  
0020  84 85 08 00 7a f4 00 00 00 0b 7d                  ....z.....}  
  
0000  00 50 56 fc 4f b3 00 0c 29 b7 fb 48 08 00 45 00   .PV.O...)..H..E.  
0010  00 1d 7d 40 00 00 40 01 23 d9 c0 a8 84 85 64 58   ..}@..@.#.....dX  
0020  30 41 00 00 82 f4 00 00 00 0b 7d 00 00 00 00 00   0A........}.....  
0030  00 00 00 00 00 00 00 00 00 00 00 00               ............
```

I put the output into [cyberchef](https://gchq.github.io/CyberChef/#recipe=From_Hexdump()&input=MDAwMCAgMDAgMGMgMjkgYjcgZmIgNDggMDAgNTAgNTYgYzAgMDAgMDggMDggMDAgNDUgMDAgICAuLikuLkguUFYuLi4uLkUuCjAwMTAgIDAwIDIwIDAwIDAxIDAwIDAwIDQwIDAxIDcwIDBmIDU5IDZlIDZjIDMxIGMwIGE4ICAgLiAuLi4uQC5wLllubDEuLgowMDIwICA4NCA4NSAwOCAwMCAyMCAyMyAwMCAwMCAwMCAwMCA2MiA3OSA3NSA2MyAgICAgICAgIC4uLi4gIy4uLi5ieXVjCgowMDAwICAwMCA1MCA1NiBmYyA0ZiBiMyAwMCAwYyAyOSBiNyBmYiA0OCAwOCAwMCA0NSAwMCAgIC5QVi5PLi4uKS4uSC4uRS4KMDAxMCAgMDAgMjAgYzggN2YgMDAgMDAgNDAgMDEgYTcgOTAgYzAgYTggODQgODUgNTkgNmUgICAuIC4uLi5ALi4uLi4uLlluCjAwMjAgIDZjIDMxIDAwIDAwIDI4IDIzIDAwIDAwIDAwIDAwIDYyIDc5IDc1IDYzIDAwIDAwICAgbDEuLigjLi4uLmJ5dWMuLgowMDMwICAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAgICAgICAgICAgICAgIC4uLi4uLi4uLi4uLgoKMDAwMCAgMDAgMGMgMjkgYjcgZmIgNDggMDAgNTAgNTYgYzAgMDAgMDggMDggMDAgNDUgMDAgICAuLikuLkguUFYuLi4uLkUuCjAwMTAgIDAwIDIwIDAwIDAxIDAwIDAwIDQwIDAxIDhhIDBlIDU5IDMzIDUyIDZkIGMwIGE4ICAgLiAuLi4uQC4uLlkzUm0uLgowMDIwICA4NCA4NSAwOCAwMCAwOCA0NCAwMCAwMCAwMCAwMSA3NCA2NiA3YiA1NCAgICAgICAgIC4uLi4uRC4uLi50ZntUCgowMDAwICAwMCA1MCA1NiBmYyA0ZiBiMyAwMCAwYyAyOSBiNyBmYiA0OCAwOCAwMCA0NSAwMCAgIC5QVi5PLi4uKS4uSC4uRS4KMDAxMCAgMDAgMjAgMjEgNmMgMDAgMDAgNDAgMDEgNjggYTMgYzAgYTggODQgODUgNTkgMzMgICAuICFsLi5ALmguLi4uLlkzCjAwMjAgIDUyIDZkIDAwIDAwIDEwIDQ0IDAwIDAwIDAwIDAxIDc0IDY2IDdiIDU0IDAwIDAwICAgUm0uLi5ELi4uLnRme1QuLgowMDMwICAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAgICAgICAgICAgICAgIC4uLi4uLi4uLi4uLgoKMDAwMCAgMDAgMGMgMjkgYjcgZmIgNDggMDAgNTAgNTYgYzAgMDAgMDggMDggMDAgNDUgMDAgICAuLikuLkguUFYuLi4uLkUuCjAwMTAgIDAwIDIwIDAwIDAxIDAwIDAwIDQwIDAxIDdlIDBlIDY1IDMxIDUyIDZmIGMwIGE4ICAgLiAuLi4uQC5%2BLmUxUm8uLgowMDIwICA4NCA4NSAwOCAwMCAxMCA1OCAwMCAwMCAwMCAwMiA3NSA3MiA3MiAzMyAgICAgICAgIC4uLi4uWC4uLi51cnIzCgowMDAwICAwMCA1MCA1NiBmYyA0ZiBiMyAwMCAwYyAyOSBiNyBmYiA0OCAwOCAwMCA0NSAwMCAgIC5QVi5PLi4uKS4uSC4uRS4KMDAxMCAgMDAgMjAgZWEgMzQgMDAgMDAgNDAgMDEgOTMgZGEgYzAgYTggODQgODUgNjUgMzEgICAuIC40Li5ALi4uLi4uLmUxCjAwMjAgIDUyIDZmIDAwIDAwIDE4IDU4IDAwIDAwIDAwIDAyIDc1IDcyIDcyIDMzIDAwIDAwICAgUm8uLi5YLi4uLnVycjMuLgowMDMwICAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAgICAgICAgICAgICAgIC4uLi4uLi4uLi4uLgoKMDAwMCAgMDAgMGMgMjkgYjcgZmIgNDggMDAgNTAgNTYgYzAgMDAgMDggMDggMDAgNDUgMDAgICAuLikuLkguUFYuLi4uLkUuCjAwMTAgIDAwIDIwIDAwIDAxIDAwIDAwIDQwIDAxIGFmIDJjIDRkIDMxIDM5IDUxIGMwIGE4ICAgLiAuLi4uQC4uLE0xOVEuLgowMDIwICA4NCA4NSAwOCAwMCAzMSA2YSAwMCAwMCAwMCAwMyA3NCA1ZiA1MiAzMyAgICAgICAgIC4uLi4xai4uLi50X1IzCgowMDAwICAwMCA1MCA1NiBmYyA0ZiBiMyAwMCAwYyAyOSBiNyBmYiA0OCAwOCAwMCA0NSAwMCAgIC5QVi5PLi4uKS4uSC4uRS4KMDAxMCAgMDAgMjAgMjYgNzMgMDAgMDAgNDAgMDEgODggYmEgYzAgYTggODQgODUgNGQgMzEgICAuICZzLi5ALi4uLi4uLk0xCjAwMjAgIDM5IDUxIDAwIDAwIDM5IDZhIDAwIDAwIDAwIDAzIDc0IDVmIDUyIDMzIDAwIDAwICAgOVEuLjlqLi4uLnRfUjMuLgowMDMwICAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAgICAgICAgICAgICAgIC4uLi4uLi4uLi4uLgoKMDAwMCAgMDAgMGMgMjkgYjcgZmIgNDggMDAgNTAgNTYgYzAgMDAgMDggMDggMDAgNDUgMDAgICAuLikuLkguUFYuLi4uLkUuCjAwMTAgIDAwIDIwIDAwIDAxIDAwIDAwIDQwIDAxIDlkIDM2IDRlIDQ4IDRhIDMwIGMwIGE4ICAgLiAuLi4uQC4uNk5ISjAuLgowMDIwICA4NCA4NSAwOCAwMCAyNiA1OCAwMCAwMCAwMCAwNCA2NCAzMyA2ZCA3MCAgICAgICAgIC4uLi4mWC4uLi5kM21wCgowMDAwICAwMCA1MCA1NiBmYyA0ZiBiMyAwMCAwYyAyOSBiNyBmYiA0OCAwOCAwMCA0NSAwMCAgIC5QVi5PLi4uKS4uSC4uRS4KMDAxMCAgMDAgMjAgNjYgMjUgMDAgMDAgNDAgMDEgMzcgMTIgYzAgYTggODQgODUgNGUgNDggICAuIGYlLi5ALjcuLi4uLk5ICjAwMjAgIDRhIDMwIDAwIDAwIDJlIDU4IDAwIDAwIDAwIDA0IDY0IDMzIDZkIDcwIDAwIDAwICAgSjAuLi5YLi4uLmQzbXAuLgowMDMwICAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAgICAgICAgICAgICAgIC4uLi4uLi4uLi4uLgoKMDAwMCAgMDAgMGMgMjkgYjcgZmIgNDggMDAgNTAgNTYgYzAgMDAgMDggMDggMDAgNDUgMDAgICAuLikuLkguUFYuLi4uLkUuCjAwMTAgIDAwIDIwIDAwIDAxIDAwIDAwIDQwIDAxIDc5IDBlIDU4IDMxIDY0IDZmIGMwIGE4ICAgLiAuLi4uQC55LlgxZG8uLgowMDIwICA4NCA4NSAwOCAwMCA1MyA2YiAwMCAwMCAwMCAwNSA3NCAyMSAzMCA2ZSAgICAgICAgIC4uLi5Tay4uLi50ITBuCgowMDAwICAwMCA1MCA1NiBmYyA0ZiBiMyAwMCAwYyAyOSBiNyBmYiA0OCAwOCAwMCA0NSAwMCAgIC5QVi5PLi4uKS4uSC4uRS4KMDAxMCAgMDAgMjAgZjcgNzkgMDAgMDAgNDAgMDEgODEgOTUgYzAgYTggODQgODUgNTggMzEgICAuIC55Li5ALi4uLi4uLlgxCjAwMjAgIDY0IDZmIDAwIDAwIDViIDZiIDAwIDAwIDAwIDA1IDc0IDIxIDMwIDZlIDAwIDAwICAgZG8uLltrLi4uLnQhMG4uLgowMDMwICAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAgICAgICAgICAgICAgIC4uLi4uLi4uLi4uLgoKMDAwMCAgMDAgMGMgMjkgYjcgZmIgNDggMDAgNTAgNTYgYzAgMDAgMDggMDggMDAgNDUgMDAgICAuLikuLkguUFYuLi4uLkUuCjAwMTAgIDAwIDIwIDAwIDAxIDAwIDAwIDQwIDAxIDlmIDAxIDRkIDMzIDQ5IDdhIGMwIGE4ICAgLiAuLi4uQC4uLk0zSXouLgowMDIwICA4NCA4NSAwOCAwMCA3NyAzZiAwMCAwMCAwMCAwNiA1ZiA0YyAyMSA2ZSAgICAgICAgIC4uLi53Py4uLi5fTCFuCgowMDAwICAwMCA1MCA1NiBmYyA0ZiBiMyAwMCAwYyAyOSBiNyBmYiA0OCAwOCAwMCA0NSAwMCAgIC5QVi5PLi4uKS4uSC4uRS4KMDAxMCAgMDAgMjAgMzUgYzMgMDAgMDAgNDAgMDEgNjkgM2YgYzAgYTggODQgODUgNGQgMzMgICAuIDUuLi5ALmk/Li4uLk0zCjAwMjAgIDQ5IDdhIDAwIDAwIDdmIDNmIDAwIDAwIDAwIDA2IDVmIDRjIDIxIDZlIDAwIDAwICAgSXouLi4/Li4uLl9MIW4uLgowMDMwICAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAgICAgICAgICAgICAgIC4uLi4uLi4uLi4uLgoKMDAwMCAgMDAgMGMgMjkgYjcgZmIgNDggMDAgNTAgNTYgYzAgMDAgMDggMDggMDAgNDUgMDAgICAuLikuLkguUFYuLi4uLkUuCjAwMTAgIDAwIDIwIDAwIDAxIDAwIDAwIDQwIDAxIDc2IDA0IDU4IDMwIDY3IDdhIGMwIGE4ICAgLiAuLi4uQC52LlgwZ3ouLgowMDIwICA4NCA4NSAwOCAwMCA2NSA1MSAwMCAwMCAwMCAwNyAzMyA3MyA1ZiAzNCAgICAgICAgIC4uLi5lUS4uLi4zc180CgowMDAwICAwMCA1MCA1NiBmYyA0ZiBiMyAwMCAwYyAyOSBiNyBmYiA0OCAwOCAwMCA0NSAwMCAgIC5QVi5PLi4uKS4uSC4uRS4KMDAxMCAgMDAgMjAgNzIgMjIgMDAgMDAgNDAgMDEgMDMgZTMgYzAgYTggODQgODUgNTggMzAgICAuIHIiLi5ALi4uLi4uLlgwCjAwMjAgIDY3IDdhIDAwIDAwIDZkIDUxIDAwIDAwIDAwIDA3IDMzIDczIDVmIDM0IDAwIDAwICAgZ3ouLm1RLi4uLjNzXzQuLgowMDMwICAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAgICAgICAgICAgICAgIC4uLi4uLi4uLi4uLgoKMDAwMCAgMDAgMGMgMjkgYjcgZmIgNDggMDAgNTAgNTYgYzAgMDAgMDggMDggMDAgNDUgMDAgICAuLikuLkguUFYuLi4uLkUuCjAwMTAgIDAwIDIwIDAwIDAxIDAwIDAwIDQwIDAxIDZhIDE2IDU4IDMwIDczIDY4IGMwIGE4ICAgLiAuLi4uQC5qLlgwc2guLgowMDIwICA4NCA4NSAwOCAwMCAyNiA3NiAwMCAwMCAwMCAwOCA3MiAzMyA1ZiA0ZSAgICAgICAgIC4uLi4mdi4uLi5yM19OCgowMDAwICAwMCA1MCA1NiBmYyA0ZiBiMyAwMCAwYyAyOSBiNyBmYiA0OCAwOCAwMCA0NSAwMCAgIC5QVi5PLi4uKS4uSC4uRS4KMDAxMCAgMDAgMjAgZmUgNjcgMDAgMDAgNDAgMDEgNmIgYWYgYzAgYTggODQgODUgNTggMzAgICAuIC5nLi5ALmsuLi4uLlgwCjAwMjAgIDczIDY4IDAwIDAwIDJlIDc2IDAwIDAwIDAwIDA4IDcyIDMzIDVmIDRlIDAwIDAwICAgc2guLi52Li4uLnIzX04uLgowMDMwICAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAgICAgICAgICAgICAgIC4uLi4uLi4uLi4uLgoKMDAwMCAgMDAgMGMgMjkgYjcgZmIgNDggMDAgNTAgNTYgYzAgMDAgMDggMDggMDAgNDUgMDAgICAuLikuLkguUFYuLi4uLkUuCjAwMTAgIDAwIDIwIDAwIDAxIDAwIDAwIDQwIDAxIDVhIGVkIDYyIDQ3IDc4IDdhIGMwIGE4ICAgLiAuLi4uQC5aLmJHeHouLgowMDIwICA4NCA4NSAwOCAwMCA2OCAzMCAwMCAwMCAwMCAwOSAzMCA3NCA1ZiA1MiAgICAgICAgIC4uLi5oMC4uLi4wdF9SCgowMDAwICAwMCA1MCA1NiBmYyA0ZiBiMyAwMCAwYyAyOSBiNyBmYiA0OCAwOCAwMCA0NSAwMCAgIC5QVi5PLi4uKS4uSC4uRS4KMDAxMCAgMDAgMjAgZWMgZWMgMDAgMDAgNDAgMDEgNmUgMDEgYzAgYTggODQgODUgNjIgNDcgICAuIC4uLi5ALm4uLi4uLmJHCjAwMjAgIDc4IDdhIDAwIDAwIDcwIDMwIDAwIDAwIDAwIDA5IDMwIDc0IDVmIDUyIDAwIDAwICAgeHouLnAwLi4uLjB0X1IuLgowMDMwICAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAgICAgICAgICAgICAgIC4uLi4uLi4uLi4uLgoKMDAwMCAgMDAgMGMgMjkgYjcgZmIgNDggMDAgNTAgNTYgYzAgMDAgMDggMDggMDAgNDUgMDAgICAuLikuLkguUFYuLi4uLkUuCjAwMTAgIDAwIDIwIDAwIDAxIDAwIDAwIDQwIDAxIDcyIDA2IDU4IDMxIDZiIDc3IGMwIGE4ICAgLiAuLi4uQC5yLlgxa3cuLgowMDIwICA4NCA4NSAwOCAwMCBhMyAxZSAwMCAwMCAwMCAwYSAyMSA2NCAzMyA3MyAgICAgICAgIC4uLi4uLi4uLi4hZDNzCgowMDAwICAwMCA1MCA1NiBmYyA0ZiBiMyAwMCAwYyAyOSBiNyBmYiA0OCAwOCAwMCA0NSAwMCAgIC5QVi5PLi4uKS4uSC4uRS4KMDAxMCAgMDAgMjAgNmMgMmEgMDAgMDAgNDAgMDEgMDUgZGQgYzAgYTggODQgODUgNTggMzEgICAuIGwqLi5ALi4uLi4uLlgxCjAwMjAgIDZiIDc3IDAwIDAwIGFiIDFlIDAwIDAwIDAwIDBhIDIxIDY0IDMzIDczIDAwIDAwICAga3cuLi4uLi4uLiFkM3MuLgowMDMwICAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAgICAgICAgICAgICAgIC4uLi4uLi4uLi4uLgoKMDAwMCAgMDAgMGMgMjkgYjcgZmIgNDggMDAgNTAgNTYgYzAgMDAgMDggMDggMDAgNDUgMDAgICAuLikuLkguUFYuLi4uLkUuCjAwMTAgIDAwIDFkIDAwIDAxIDAwIDAwIDQwIDAxIGExIDE4IDY0IDU4IDMwIDQxIGMwIGE4ICAgLi4uLi4uQC4uLmRYMEEuLgowMDIwICA4NCA4NSAwOCAwMCA3YSBmNCAwMCAwMCAwMCAwYiA3ZCAgICAgICAgICAgICAgICAgIC4uLi56Li4uLi59CgowMDAwICAwMCA1MCA1NiBmYyA0ZiBiMyAwMCAwYyAyOSBiNyBmYiA0OCAwOCAwMCA0NSAwMCAgIC5QVi5PLi4uKS4uSC4uRS4KMDAxMCAgMDAgMWQgN2QgNDAgMDAgMDAgNDAgMDEgMjMgZDkgYzAgYTggODQgODUgNjQgNTggICAuLn1ALi5ALiMuLi4uLmRYCjAwMjAgIDMwIDQxIDAwIDAwIDgyIGY0IDAwIDAwIDAwIDBiIDdkIDAwIDAwIDAwIDAwIDAwICAgMEEuLi4uLi4uLn0uLi4uLgowMDMwICAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAwMCAgICAgICAgICAgICAgIC4uLi4uLi4uLi4uLg&oeol=FF) and did the recipe `from hexdump`. Then I just typed out the flag!

`byuctf{Turr3t_R3d3mpt!0n_L!n3s_4r3_N0t_R!d3s}`

___

## Alright. Time Paradox

**Category**: Forensics

**Author**: JC6143

**Description**: To maintain a constant testing cycle, I simulate daylight at all hours and add adrenal vapor to your oxygen supply. So you may be confused about the passage of time. The point is, yesterday was your birthday. I thought you'd want to know.

*The pcap file contains a flag for each of the following challenges: "There will be cake", "Are you still there?", "Alright. Paradox time", and "Corrupted Cores".  
If the flag you found doesn't work, then it most likely belongs to one of the other 3 challenges.

Hint: What protocol is associated with time?

In wireshark I click on an `NTP` packet and follow UDP stream 0:
```shell
#.
.............eS.b....eS.y....eS.u....eS.c....#.
.............eS.t....eS.f....eS.{....eS.S....#.
.............eS.0....eS._....eS.M....eS.y....#.
.............eS._....eS.P....eS.4....eS.r....#.
.............eS.4....eS.d....eS.0....eS.x....#.
.............eS._....eS.!....eS.d....eS.3....#.
.............eS.4....eS._....eS.D....eS.!....#.
.............eS.d....eS.n....eS.t....eS._....#.
.............eS.W....eS.0....eS.r....eS.k....#.
.............eS.}....eS......eS......eS......
```

I delete the `.............eS.` sections and get the flag.

`byuctf{S0_My_P4r4d0x_!d34_D!dnt_W0rk}`

___

## Corrupted Cores

**Category**: Forensics

**Author**: JC6143

**Description**: "The scientists were always hanging cores on me to regulate my behavior. I've heard voices all my life. But now I hear the voice of a conscience, and it's terrifying, because for the first time it's my voice."

*The pcap file contains a flag for each of the following challenges: "There will be cake", "Are you still there?", "Alright. Paradox time", and "Corrupted Cores".  
If the flag you found doesn't work, then it most likely belongs to one of the other 3 challenges.

Hint: the voices may not belong to a single identity Hint2: the arp packets are not part of this challenge.

I pull the IP addresses from the ICMP packets:
```shell
tshark -r GLaDOS_Network.pcapng -Y icmp -T fields -e ip.dst  
192.168.132.133  
89.110.108.49  
192.168.132.133  
89.51.82.109  
192.168.132.133  
101.49.82.111  
192.168.132.133  
77.49.57.81  
192.168.132.133  
78.72.74.48  
192.168.132.133  
88.49.100.111  
192.168.132.133  
77.51.73.122  
192.168.132.133  
88.48.103.122  
192.168.132.133  
88.48.115.104  
192.168.132.133  
98.71.120.122  
192.168.132.133  
88.49.107.119  
192.168.132.133  
100.88.48.65
```

I need to map the non local IPs to ascii and concatenate them:
```shell
tshark -r GLaDOS_Network.pcapng -Y icmp -T fields -e ip.dst |  
grep -v '^192\.168\.132\.133$' |  
awk -F. '{printf "%c%c%c%c",$1,$2,$3,$4} END{print ""}'  
Ynl1Y3Rme1RoM19QNHJ0X1doM3IzX0gzX0shbGxzX1kwdX0A  
```

Just base64 decode:
```shell
echo 'Ynl1Y3Rme1RoM19QNHJ0X1doM3IzX0gzX0shbGxzX1kwdX0A' | base64 -d  
byuctf{Th3_P4rt_Wh3r3_H3_K!lls_Y0u}
```

`byuctf{Th3_P4rt_Wh3r3_H3_K!lls_Y0u}`
