---
layout: post
title:  "Forensics Challenges"
date:   2026-10-12 12:51:00 -0400
author: uzi
tags: [CTF]
permalink: /kubstu-ctf-2026-forensics/
---
* TOC
{:toc}

## Baby forensics

**Category**: forensics

**Description**: While the employee was distracted by a meeting, you obtained an image of their workstation. Your task is to reconstruct the infection chain, determine what exactly happened, and answer questions related to the incident. While the employee was distracted by a meeting, you obtained an image of their workstation. Your task is to reconstruct the chain of infection, determine exactly what happened, and answer questions regarding the incident.

1. What did the user download?
2. What is the address of the C2 server? (including the port)
3. What software was used to implement the persistence mechanism? 
4. Flag format: `KubSTU{text.txt|184.0.0.2:1234|abob.exe}`

I get the challenge files:
```shell
unzip workstation23.zip  
Archive:  workstation23.zip  
  inflating: WORKSTATION-23/C/$Recycle.Bin/S-1-5-21-1961432191-301355982-4121230938-1001/$I8O5VA5.exe.txt  
  inflating: WORKSTATION-23/C/$Recycle.Bin/S-1-5-21-1961432191-301355982-4121230938-1001/$R8O5VA5.exe  
  inflating: WORKSTATION-23/C/Program Files/Google/Chrome/Application/chrome.exe 
  inflating: WORKSTATION-23/C/Program Files/Microsoft OneDrive/OneDrive.exe  
  inflating: WORKSTATION-23/C/ProgramData/.cache/agent_state.dat  
  inflating: WORKSTATION-23/C/ProgramData/AcmeBackup/backup_job.log  
  inflating: WORKSTATION-23/C/ProgramData/AcmeBackup/config.ini  
  inflating: WORKSTATION-23/C/ProgramData/Microsoft/Windows/Start 

... etc ...

  inflating: WORKSTATION-23/C/Windows/System32/config/SOFTWARE.txt  
  inflating: WORKSTATION-23/C/Windows/System32/config/SYSTEM.txt  
  inflating: WORKSTATION-23/C/Windows/System32/drivers/etc/hosts  
  inflating: WORKSTATION-23/C/Windows/System32/winevt/Logs/README.txt  
  inflating: WORKSTATION-23/C/Windows/Temp/edge_update.log
```

I found the download here:
```shell
sqlite3 'WORKSTATION-23/C/Users/Alex/AppData/Local/Google/Chrome/User Data/Default/History'  
SQLite version 3.46.1 2024-08-13 09:16:08  
Enter ".help" for usage hints.  
sqlite> .tables  
urls    visits  
sqlite> select * from urls;  
1|https://intranet.contoso-training.example/|Contoso Training — Home|1|0|13425641820000000|0  
2|https://mail.contoso-training.example/|Mail — Alex M.|1|0|13425642189000000|0  
3|https://supplier-portal.example/invoices|Supplier Portal|1|0|13425642764000000|0  
4|http://198.51.100.23:8080/InvoiceViewer.exe|Invoice Viewer — Download|1|0|13425642871000000|0  
5|https://support.microsoft.com/windows|Windows help & learning|1|0|13425643207000000|0
```

They downloaded `InvoiceViewer.exe` from `http://198.51.100.23:8080`.

I found the c2 here:
```shell
cat WORKSTATION-23/C/ProgramData/.cache/agent_state.dat  
[sync]  
profile=onedrive-update  
transport=tcp  
endpoint=198.51.100.23:4444  
last_status=connected  
note=Documentation-only TEST-NET endpoint; no network activity is possible.
```

The c2 is `198.51.100.23:4444`.

I found the persistance mechanism here:
```shell
cat 'WORKSTATION-23/C/Users/Alex/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Startup/OneDriveUpdate.vbs'  
' INERT TRAINING ARTIFACT — comments only; does not execute instructions.  
' Claimed task name: OneDriveUpdate  
' Intended target recorded in related state: %APPDATA%\Microsoft\OneDrive\RuntimeBroker.exe  
' The file is placed in the Startup directory to model persistence evidence.
```

`KubSTU{InvoiceViewer.exe|198.51.100.23:4444|OneDriveUpdate.vbs}`

___

## Echoes of the PLC

**Category**: forensics

**Description**: After the nightly maintenance of the “smart building” laboratory, the controller calibration log disappeared. The network dump contains a mix of monitoring, updates, equipment management, and large‑scale backups. The analyst is sure that the log was transmitted via one of the service channels, but the usual data search yielded nothing. Flag format: `KubSTU{...}`

I get the challenge file:
```shell
file echoes_of_the_plc.pcap  
echoes_of_the_plc.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 65535)
```

I opened the pcap in wireshark. There was about 33 thousand packets. 

I saw these protocols:
```shell
Modbus, MQTT, CoAP, syslog, SNMP, NTP, TFTP, DHCP, ICMP and ARP.
```

I searched for `frame contains "calibration"` and I found an MQTT message for `facility/maintenance/calibration-manifest`:
```shell
{"source":"Modbus/TCP unit=7 function=0x04","order":"transaction_id ascending","mask":"ICMP Echo Reply id=3199; sequence = transaction_id - 0x4000","final":"zlib"}
```

This tells me how to rebuild the log by pulling data from modbus. I need to take only the responses from unit 7 with function 4. There are 9, with transaction IDs `0x4001` to `0x4009`. Then I need to take the echo replies with id 3199. 

For each chunk, XOR the modbus data with the ICMP payload whose sequence is the transaction ID minus `0x4000`. Concatenate the results in order and it will be a zlib compressed file. 

I ran this python to get the flag:
```python
python3  
Python 3.13.5 (main, Aug 10 2026, 12:06:59) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> #!/usr/bin/env python3  
... # pip install scapy  
... import sys, zlib  
... from scapy.all import rdpcap, TCP, ICMP, Raw  
...  
... pkts = rdpcap(sys.argv[1] if len(sys.argv) > 1 else "echoes_of_the_plc.pcap")  
...  
... modbus, mask = {}, {}  
... for p in pkts:  
...     if p.haslayer(ICMP) and p[ICMP].type == 0 and p[ICMP].id == 3199:  
...         mask[p[ICMP].seq] = bytes(p[ICMP].payload)      # echo reply masks  
...     elif p.haslayer(TCP) and p[TCP].sport == 502 and p.haslayer(Raw):  
...         d = bytes(p[Raw])  
...         if len(d) > 8 and d[6] == 7 and d[7] == 4:      # unit 7, read input regs  
...             modbus[int.from_bytes(d[:2], "big")] = d[9:9 + d[8]]  
...  
... data = b"".join(  
...     bytes(a ^ b for a, b in zip(modbus[t], mask[t - 0x4000])) for t in sorted(modbus)  
... )  
... print(zlib.decompress(data).decode())  
...  
PLC calibration journal  
device=west-wing-controller-07  
maintenance-window=2026-08-15T10:37:00Z  
Flag: KubSTU{modbus_echoes_keep_the_secret}
```

`KubSTU{modbus_echoes_keep_the_secret}`

___

## Timing

**Category**: forensics

**Description**: One of the temporary exports has disappeared from the portable storage device in the materials preparation department. The employee claims that he managed to copy the results before leaving and cleared the workspace, but the transfer log appears to be incomplete. Flag format: `KubSTU{...}`

I get the challenge file:
```shell
file last_export.img  
last_export.img: DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "mkfs.fat", sectors/cluster 4, reserved sectors 4, root entries 512, Media descriptor 0xf8, sectors/FAT 96, sectors/track 32, heads 8, sectors 98304 (volumes > 32 MB), serial number 0xc5760c28, label: "LASTEXPORT ", FAT (16 bit)
```

I check the filesystem with `fatcat`:
```shell
fatcat last_export.img -l / -d  
Listing path /  
Directory cluster: 0  
d 15/8/2026 12:32:22  Archive/ (ARCHIVE)                                 c=2  
d 15/8/2026 12:32:22  Logs/ (LOGS)                                       c=3  
d 15/8/2026 12:32:22  Temp/ (TEMP)                                       c=4  
f 15/8/2026 12:32:22  README.TXT                                         c=5 s=492 (492B)  
f 15/8/2026 12:32:22  sync_7D3.tmp (YNC_7D3.TMP)                         c=16 s=14848 (14.5K) d  

fatcat last_export.img -l Archive/ -d  
Listing path Archive/  
Directory cluster: 2  
d 15/8/2026 12:32:22  ./ (.)                                             c=2  
d 15/8/2026 12:32:22  ../ (..)                                           c=0  
f 15/8/2026 12:32:22  SUMMARY.TXT                                        c=6 s=154 (154B)  

fatcat last_export.img -l Logs/ -d  
Listing path Logs/  
Directory cluster: 3  
d 15/8/2026 12:32:22  ./ (.)                                             c=3  
d 15/8/2026 12:32:22  ../ (..)                                           c=0  
f 15/8/2026 12:32:22  TRANSFER.LOG                                       c=7 s=418 (418B)  

fatcat last_export.img -l Temp/ -d  
Listing path Temp/  
Directory cluster: 4  
d 15/8/2026 12:32:22  ./ (.)                                             c=4  
d 15/8/2026 12:32:22  ../ (..)                                           c=0  
f 15/8/2026 12:32:22  CACHE.BIN                                          c=8 s=8192 (8K)  
f 15/8/2026 12:32:22  draft_notes.tmp (RAFT_~1.TMP)                      c=12 s=70 (70B) d  
f 15/8/2026 12:32:22  unused_ledger.tmp (NUSED~1.TMP)                    c=13 s=35 (35B) d  
f 15/8/2026 12:32:22  PACKED.TMP                                         c=14 s=3076 (3.00391K)
```

There was 3 deleted files:
```shell
sync_7D3.tmp
draft_notes.tmp
unused_ledger.tmp
```

Recover the files:
```shell
fatcat last_export.img -R 16 -s 14848 > sync_7D3.tmp  

fatcat last_export.img -R 12 -s 70 > draft_notes.tmp  

fatcat last_export.img -R 13 -s 35 > unused_ledger.tmp
```

Two text files and a windows executable:
```shell
file sync_7D3.tmp draft_notes.tmp unused_ledger.tmp  
sync_7D3.tmp:      PE32 executable for MS Windows 4.00 (console), Intel i386 (stripped to external PDB), 9 sections  
draft_notes.tmp:   ASCII text, with CRLF line terminators  
unused_ledger.tmp: ASCII text, with CRLF line terminators
```

Text files:
```shell
cat draft_notes.tmp  
port check complete  
old exporter should be retired before September  

cat unused_ledger.tmp  
ref: 7D2, 7D1  
no pending exports
```

I put the binary into ghidra. I found this function:
```shell
undefined4 FUN_00402590(void)

{
  undefined *puVar1;
  FILE *pFVar2;
  int iVar3;
  int iVar4;
  undefined4 *puVar5;
  byte *pbVar6;
  undefined4 *puVar7;
  byte local_5c [16];
  undefined4 local_4c;
  undefined4 local_3c;
  undefined4 local_38;
  undefined4 auStack_36 [4];
  undefined4 local_26 [4];
  undefined4 local_16;
  undefined2 local_12;
  
  FUN_00401580();
  puVar1 = PTR_FUN_00403020;
  iVar3 = 0;
  do {
    iVar4 = iVar3 + 1;
    local_5c[iVar3] = (char)iVar3 * '#' + 0x51U ^ (&DAT_00404058)[iVar3];
    iVar3 = iVar4;
  } while (iVar4 != 0x13);
  iVar3 = 0;
  do {
    *(byte *)((int)&local_4c + iVar3 + 3) = (char)iVar3 * '#' - 0x16U ^ (&DAT_00404044)[iVar3];
    iVar3 = iVar3 + 1;
  } while (iVar3 != 0x13);
  puVar5 = &local_4c;
  puVar7 = (undefined4 *)((int)register0x00000010 + -0x36);
  for (iVar3 = 4; iVar3 != 0; iVar3 = iVar3 + -1) {
    *puVar7 = *puVar5;
    puVar5 = puVar5 + 1;
    puVar7 = puVar7 + 1;
  }
  pbVar6 = local_5c;
  puVar5 = local_26;
  for (iVar3 = 4; iVar3 != 0; iVar3 = iVar3 + -1) {
    *puVar5 = *(undefined4 *)pbVar6;
    pbVar6 = pbVar6 + 4;
    puVar5 = puVar5 + 1;
  }
  local_16 = local_3c;
  local_12 = (undefined2)local_38;
  pFVar2 = (FILE *)(*(code *)PTR_FUN_00403020)(1);
  fwrite((undefined4 *)((int)register0x00000010 + -0x36),1,0x26,pFVar2);
  pFVar2 = (FILE *)(*(code *)puVar1)(1);
  fputc(10,pFVar2);
  return 0;
}
```
Extract the two XOR tables from the data section to recover the flag.

I ran this python to get the flag:
```python
python3  
Python 3.13.5 (main, Aug 10 2026, 12:06:59) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import pefile  
...  
... pe = pefile.PE('sync_7D3.tmp')  
... ib = pe.OPTIONAL_HEADER.ImageBase  
...  
... def rd(va, n):  
...     return pe.get_data(va - ib, n)  
...  
... t1 = rd(0x404058, 19)  
... t2 = rd(0x404044, 19)  
...  
... a = bytes(((i * 0x23 + 0x51) & 0xff) ^ t1[i] for i in range(19))  
... b = bytes(((i * 0x23 - 0x16) & 0xff) ^ t2[i] for i in range(19))  
...  
... flag = a[16:19] + b[0:13] + a[0:16] + b[13:19]  
... print(flag.decode())  
...  
KubSTU{aU70p5y_D3l373d_P3_4nd_r3v3r53}
```

`KubSTU{aU70p5y_D3l373d_P3_4nd_r3v3r53}`
