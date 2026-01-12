---
layout: post
title:  "Stuck In The Middle With You"
date:   2026-01-12 01:04:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /scarletCTF-stuck-in-the-middle-with-you/
---

**Title**: Stuck In The Middle With You

**Category**: OSINT

**Author**: glqce

**Description**: We're trying to figure out how to track this Tor traffic but all we've got is this string, `A68097FE97D3065B1A6F4CE7187D753F8B8513F5`! We don't know what to do with it. We're looking for someone responsible for hosting multiple nodes. Can you find the IPv4 addresses this node and any of its effective family members?

FLAG FORMAT: `RUSEC{family_ip1:family_ip2:...:family_ipX}` for X family members

The flag will be the IPs of the node and all the associated family members **in order of oldest node to youngest**, based on when they were first seen, separated by colons.

For example, if you find five family members your flag might look something like: `RUSEC{8.8.8.8:1.1.1.1:127.0.0.1:192.168.1.1:4.2.2.2}`

`8.8.8.8` is the oldest node, then the nodes progressively get younger until we reach the youngest, `2.2.2.2`

I can query the tor projects onionoo API like this:
```shell
curl https://onionoo.torproject.org/details?lookup=A68097FE97D3065B1A6F4CE7187D753F8B8513F5 | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1408  100  1408    0     0   3664      0 --:--:-- --:--:-- --:--:--  3666
{
  "version": "8.0",
  "build_revision": "d2c1261",
  "relays_published": "2026-01-12 05:00:00",
  "relays": [
    {
      "nickname": "olabobamanmu",
      "fingerprint": "A68097FE97D3065B1A6F4CE7187D753F8B8513F5",
      "or_addresses": [
        "51.15.40.38:9001",
        "[2001:bc8:1640:1e30:dc00:ff:fe2a:2549]:9001"
      ],
      "last_seen": "2026-01-12 05:00:00",
      "last_changed_address_or_port": "2024-07-03 14:00:00",
      "first_seen": "2020-04-03 00:00:00",
      "running": true,
      "flags": [
        "Fast",
        "HSDir",
        "Running",
        "Stable",
        "V2Dir",
        "Valid"
      ],
      "country": "nl",
      "country_name": "Netherlands",
      "as": "AS12876",
      "as_name": "SCALEWAY S.A.S.",
      "consensus_weight": 21000,
      "verified_host_names": [
        "olabobamanmu.giannoug.gr"
      ],
      "last_restarted": "2025-12-10 23:23:33",
      "bandwidth_rate": 1073741824,
      "bandwidth_burst": 1073741824,
      "observed_bandwidth": 18252857,
      "advertised_bandwidth": 18252857,
      "overload_general_timestamp": 1767913200000,
      "exit_policy": [
        "reject *:*"
      ],
      "exit_policy_summary": {
        "reject": [
          "1-65535"
        ]
      },
      "contact": "giannoug|__AT-|gmail{ DOt-}com",
      "platform": "Tor 0.4.8.21 on Linux",
      "version": "0.4.8.21",
      "version_status": "recommended",
      "effective_family": [
        "414E64BA607560F9D9C196A825950DC968700420",
        "A68097FE97D3065B1A6F4CE7187D753F8B8513F5",
        "B4CAFD9CBFB34EC5DAAC146920DC7DFAFE91EA20"
      ],
      "consensus_weight_fraction": 0.000101960046,
      "guard_probability": 0,
      "middle_probability": 0.00030587654,
      "exit_probability": 0,
      "recommended_version": true,
      "measured": true
    }
  ],
  "bridges_published": "2026-01-12 04:44:11",
  "bridges": []
}
```

Check each node and order the IPs by oldest to youngest:
```shell
414E64BA607560F9D9C196A825950DC968700420
A68097FE97D3065B1A6F4CE7187D753F8B8513F5
B4CAFD9CBFB34EC5DAAC146920DC7DFAFE91EA20
```

`RUSEC{212.47.233.86:51.15.40.38:151.115.73.55}`