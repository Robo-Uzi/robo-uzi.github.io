---
layout: post
title:  "Misc Challenges"
date:   2026-05-28 13:58:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /SecLeaf-Q2-CTF-2026-misc/
---
* TOC
{:toc}

## vaultcore

**Category**: misc

**Description**: We recovered a protected vault executable from an abandoned workstation. Initial analysis suggests:
- anti-debugging routines
- payload decryption
- integrity verification

Can you recover the secure access token? Flag format: `SecLeaf{}`

```shell
file vaultcore  
vaultcore: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), statically linked, no section header  

strings vaultcore | grep Sec  
SecLeaf{str1ngs_1s_4ll_y0u_n33d}
```

`SecLeaf{str1ngs_1s_4ll_y0u_n33d}`

___

## The Invoice Incident

**Category**: misc

**Description**: At 09:14 AM, a finance employee reported receiving an urgent invoice email. Shortly after, suspicious activity began on the host. You have been provided mail and endpoint logs collected during the incident. Analyze the logs carefully, determine the malicious attachment responsible for the compromise, and submit the flag.

I get the challenge files `easy_logs.csv`, `endpoint_events.log`, and `mail_gateway.log`. Looking at `mail_gateway.log`:
```shell
cat mail_gateway.log | tail  
2026-05-02 09:51:00 ALLOW from=user51@example.com to=it.rahul subject=Routine Update attachment=None  
2026-05-02 09:52:00 ALLOW from=user52@example.com to=it.rahul subject=Routine Update attachment=None  
2026-05-02 09:53:00 ALLOW from=user53@example.com to=finance.aarti subject=Routine Update attachment=None  
2026-05-02 09:54:00 ALLOW from=user54@example.com to=finance.aarti subject=Routine Update attachment=None  
2026-05-02 09:55:00 ALLOW from=user55@example.com to=sales.vikram subject=Routine Update attachment=None  
2026-05-02 09:56:00 ALLOW from=user56@example.com to=finance.aarti subject=Routine Update attachment=None  
2026-05-02 09:57:00 ALLOW from=user57@example.com to=it.rahul subject=Routine Update attachment=None  
2026-05-02 09:58:00 ALLOW from=user58@example.com to=finance.aarti subject=Routine Update attachment=None  
2026-05-02 09:59:00 ALLOW from=user59@example.com to=finance.aarti subject=Routine Update attachment=None  
2026-05-02 09:14:03 ALLOW from=billing@micr0soft-support.com to=finance.aarti subject=Pending Invoice Notice attachment=Invoice_April_2026.docm
```

There is a suspicious email coming from `billing@micr0soft-support.com`.

`SecLeaf{Invoice_April_2026.docm}`

___

## wrong_turn

**Category**: misc

**Description**: There is Secure vault which has hard coded flag in it. Decrypt the password to unlock the vault. Flag Format: `SecLeaf{}`

I get the challenge file:
```shell
file wrong_turn  
wrong_turn: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), statically linked, no section header
```

I put the binary in ghidra and find this function:
```shell
undefined8 FUN_00105528(void)

{
  int iVar1;
  undefined7 local_53;
  undefined4 uStack_4c;
  undefined local_48 [64];
  
  local_53 = 0x7365736e65706f;
  uStack_4c = 0x656d61;
  FUN_001054fc();
  FUN_00105512();
  FUN_001053d3(0x1063d1);
  FUN_001053e3(0x1063e2,local_48);
  iVar1 = FUN_001053f3(local_48,&local_53);
  if (iVar1 == 0) {
    FUN_001053c3(0x1063e7);
    FUN_001053c3(0x1063fb);
  }
  else {
    FUN_001053c3(0x10641c);
  }
  return 0;
}
```

Decode the password:
```python
python3  
Python 3.13.5 (main, May  5 2026, 21:05:52) [GCC 14.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> print((0x7365736e65706f).to_bytes(7, "little").decode())  
... print((0x656d61).to_bytes(3, "little").decode())  
...  
openses  
ame
```

```shell
./wrong_turn  
Debugger check complete...  
Decrypting secure vault...  
Enter password: opensesame  
Access granted  
SecLeaf{hardcoded_secrets_again}
```

`SecLeaf{hardcoded_secrets_again}`