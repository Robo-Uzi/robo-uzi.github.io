---
layout: post
title:  "CredVault"
date:   2026-07-21 17:49:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /omni-CTF-2026-CredVault/
---

Title: CredVault

Author: Alex_Hossu

Category: Rev

Description: PktTrack's mobile team extracted `credvault` from a rooted Android device. The binary implements `com.android.credentialservice.CredVaultService`, which issues elevated credential tokens to authorized clients. The service was recently migrated to a new Parcel format. The mobile team signed off on the migration. Nobody checked the forwarding path.

I get the challenge file:
```shell
file credvault  
credvault: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=c57da5b8bb490983b2d13cfb013e3761b5fe18c1, for GNU/Linux 3.2.0, stripped
```

I put the binary in ghidra and find this function:
```c
undefined8 FUN_00101260(int param_1,long param_2)

{
  int __fd;
  int iVar1;
  undefined8 uVar2;
  ssize_t sVar3;
  long lVar4;
  char *__src;
  ushort uVar5;
  long in_FS_OFFSET;
  undefined4 local_7c;
  sockaddr local_78;
  int local_68 [2];
  undefined1 local_60 [16];
  undefined1 local_50 [16];
  undefined8 local_40;
  long local_30;
  
  local_30 = *(long *)(in_FS_OFFSET + 0x28);
  sVar3 = read(3,s_CTF{local_test_only_not_a_real_f_00104020,0x7f);
  if (sVar3 < 1) {
    close(3);
    __src = getenv("CREDVAULT_FLAG");
    if (__src != (char *)0x0) {
      strncpy(s_CTF{local_test_only_not_a_real_f_00104020,__src,0x7f);
      DAT_0010409f = 0;
    }
  }
  else {
    lVar4 = sVar3 + -1;
    if ((&DAT_0010401f)[sVar3] != '\n') {
      lVar4 = sVar3;
    }
    s_CTF{local_test_only_not_a_real_f_00104020[lVar4] = '\0';
    close(3);
  }
  uVar5 = 0x539;
  unsetenv("CREDVAULT_FLAG");
  if (1 < param_1) {
    lVar4 = strtol(*(char **)(param_2 + 8),(char **)0x0,10);
    uVar5 = (ushort)lVar4;
  }
  __fd = socket(2,1,0);
  if (__fd < 0) {
    perror("socket");
  }
  else {
    local_7c = 1;
    setsockopt(__fd,1,2,&local_7c,4);
    local_78.sa_data[10] = '\0';
    local_78.sa_data[0xb] = '\0';
    local_78.sa_data[0xc] = '\0';
    local_78.sa_data[0xd] = '\0';
    local_78.sa_data[2] = '\0';
    local_78.sa_data[3] = '\0';
    local_78.sa_data[4] = '\0';
    local_78.sa_data[5] = '\0';
    local_78.sa_data[6] = '\0';
    local_78.sa_data[7] = '\0';
    local_78.sa_data[8] = '\0';
    local_78.sa_data[9] = '\0';
    local_78.sa_family = 2;
    local_78.sa_data._0_2_ = uVar5 << 8 | uVar5 >> 8;
    iVar1 = bind(__fd,&local_78,0x10);
    if (iVar1 < 0) {
      perror("bind");
    }
    else {
      iVar1 = listen(__fd,1);
      if (iVar1 < 0) {
        perror("listen");
      }
      else {
        iVar1 = accept(__fd,(sockaddr *)0x0,(socklen_t *)0x0);
        if (-1 < iVar1) {
          local_60 = (undefined1  [16])0x0;
          local_50 = (undefined1  [16])0x0;
          local_40 = 0;
          local_68[1] = 0xca1dcafe;
          local_68[0] = iVar1;
          FUN_00101670(local_68);
          close(iVar1);
          close(__fd);
          uVar2 = 0;
          goto LAB_001013cf;
        }
        perror("accept");
      }
    }
  }
  uVar2 = 1;
LAB_001013cf:
  if (local_30 == *(long *)(in_FS_OFFSET + 0x28)) {
    return uVar2;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}
```

And this function:
```c
bool FUN_00101520(int *param_1,uint param_2)

{
  if ((((0x17 < param_2) && (*param_1 == -0x35eeffff)) && (param_1[2] == 1)) &&
     ((param_1[3] == 0 && (param_1[4] == 0)))) {
    return (param_1[1] ^ 0xca110000U) == param_1[5];
  }
  return false;
}
```

And this one:
```c
void FUN_00101670(undefined4 *param_1)

{
  undefined4 uVar1;
  uint uVar2;
  int iVar3;
  size_t sVar4;
  undefined4 uVar5;
  long in_FS_OFFSET;
  undefined4 local_1054;
  undefined4 local_1050;
  uint local_104c;
  undefined8 local_1048;
  undefined8 uStack_1040;
  undefined8 local_1038;
  long local_40;
  
  local_40 = *(long *)(in_FS_OFFSET + 0x28);
  do {
    iVar3 = FUN_00101580(*param_1,&local_1050,8);
    uVar2 = local_104c;
    uVar1 = local_1050;
    if (iVar3 != 0) {
LAB_001018bf:
      if (local_40 == *(long *)(in_FS_OFFSET + 0x28)) {
        return;
      }
                    /* WARNING: Subroutine does not return */
      __stack_chk_fail();
    }
    uVar5 = *param_1;
    if (0x1000 < local_104c) {
      FUN_001015c0(uVar5,0xc0000002,0,0);
      goto LAB_001018bf;
    }
    if (local_104c == 0) {
      switch(local_1050) {
      case 0xca000001:
        goto switchD_0010171a_caseD_ca000001;
      case 0xca000003:
        goto switchD_0010171a_caseD_ca000003;
      case 0xca000004:
        goto switchD_0010171a_caseD_ca000004;
      case 0xca000005:
        goto switchD_0010171a_caseD_ca000005;
      }
      goto switchD_0010171a_caseD_ca000002;
    }
    iVar3 = FUN_00101580(uVar5,&local_1048,local_104c);
    if (iVar3 < 0) goto LAB_001018bf;
    switch(uVar1) {
    case 0xca000001:
      uVar5 = *param_1;
switchD_0010171a_caseD_ca000001:
      local_1054 = param_1[1];
      FUN_001015c0(uVar5,0,&local_1054,4);
      break;
    case 0xca000002:
      if (uVar2 == 0x18) {
        *(undefined8 *)(param_1 + 10) = 0;
        *(undefined8 *)(param_1 + 6) = local_1038;
        *(undefined8 *)(param_1 + 2) = local_1048;
        *(undefined8 *)(param_1 + 4) = uStack_1040;
        *(undefined8 *)(param_1 + 8) = 0x100000018;
        FUN_001015c0(*param_1,0,0,0);
        break;
      }
    default:
      uVar5 = *param_1;
switchD_0010171a_caseD_ca000002:
      FUN_001015c0(uVar5,0xc0000002,0,0);
      break;
    case 0xca000003:
      uVar5 = *param_1;
switchD_0010171a_caseD_ca000003:
      if (param_1[9] != 0) {
        iVar3 = FUN_00101520(param_1 + 2,param_1[8]);
        if (iVar3 != 0) {
          param_1[10] = 1;
          goto LAB_00101765;
        }
        param_1[10] = 0;
      }
      goto LAB_00101898;
    case 0xca000004:
      uVar5 = *param_1;
switchD_0010171a_caseD_ca000004:
      if ((param_1[10] == 0) || (iVar3 = FUN_00101560(param_1 + 2,param_1[8]), iVar3 == 0)) {
LAB_00101898:
        FUN_001015c0(uVar5,0xc0000001,0,0);
      }
      else {
        param_1[0xb] = 1;
LAB_00101765:
        FUN_001015c0(uVar5,0,0,0);
      }
      break;
    case 0xca000005:
      uVar5 = *param_1;
switchD_0010171a_caseD_ca000005:
      if (param_1[0xb] == 0) goto LAB_00101898;
      sVar4 = strlen(s_CTF{local_test_only_not_a_real_f_00104020);
      FUN_001015c0(uVar5,0,s_CTF{local_test_only_not_a_real_f_00104020,sVar4 & 0xffffffff);
    }
  } while( true );
}
```

In the network protocol, the server reads 8 bytes first:
- First 4 bytes = the command number
- Next 4 bytes = how many more bytes to read (the payload length)
In the solve script `send_cmd()` does this.

The server stores the credential as 6 integers (each 4 bytes, total 24). The solve script builds those 6 integers as:
- 0: `0xCA110001`
- 1: `0xCA110042`
- 2: `1`
- 3: `0`
- 4: `0`
- 5: `0x42`

The main loop is `FUN_00101670`. It handles four important commands:
- `0xca000002` (SET): stores the 24 byte credential sent
- `0xca000003` (AUTH): calls `FUN_00101520()` on the stored credential
- `0xca000004` (ELEVATE): first checks that youre authenticated. Then it calls `FUN_00101560()`, which checks that Word 1 (the second integer) equals `0xCA110042`.  
- `0xca000005` (GET_FLAG): checks if the elevated flag is set. If yes, it returns the flag stored in memory

Solve script:
```python
#!/usr/bin/env python3
import sys
import socket
import struct

def send_cmd(sock, cmd, data=b''):
    length = len(data)
    sock.send(struct.pack('<II', cmd, length))
    if length:
        sock.send(data)
    resp = sock.recv(8)
    if len(resp) != 8:
        raise RuntimeError('short read on response header')
    status, resp_len = struct.unpack('<II', resp)
    if resp_len:
        resp_data = sock.recv(resp_len)
    else:
        resp_data = b''
    return status, resp_data

def main():
    if len(sys.argv) >= 3:
        host = sys.argv[1]
        port = int(sys.argv[2])
    else:
        host = '127.0.0.1'
        port = 9999

    # build the valid credential
    cred = (
        struct.pack('<I', 0xCA110001) +
        struct.pack('<I', 0xCA110042) +
        struct.pack('<I', 0x00000001) +
        struct.pack('<I', 0x00000000) +
        struct.pack('<I', 0x00000000) +
        struct.pack('<I', 0x00000042)
    )
    if len(cred) != 24:
        raise RuntimeError('cred length != 24')

    sock = socket.create_connection((host, port))

    status, _ = send_cmd(sock, 0xca000002, cred)
    if status != 0:
        print(f'SET error: {status:#x}', file=sys.stderr)
        sys.exit(1)

    status, _ = send_cmd(sock, 0xca000003)
    if status != 0:
        print(f'AUTH error: {status:#x}', file=sys.stderr)
        sys.exit(1)

    status, _ = send_cmd(sock, 0xca000004)
    if status != 0:
        print(f'ELEVATE error: {status:#x}', file=sys.stderr)
        sys.exit(1)

    status, flag_data = send_cmd(sock, 0xca000005)
    if status != 0:
        print(f'FLAG error: {status:#x}', file=sys.stderr)
        sys.exit(1)

    print(flag_data.decode())

if __name__ == '__main__':
    main()
```

Encode and send the script to the challenge server:
```shell
ncat --ssl credvault-406e346b0142.inst.omnictf.com 1337  
  
╔══════════════════════════════════════════════════════════╗  
║   CREDVAULT  --  Android CredentialVault Service         ║  
║              CTF  [ Android Mobile  /  pwn ]             ║  
╚══════════════════════════════════════════════════════════╝  
  
An Android system service handles credential elevation through  
a serialized Parcel interface.  The service was recently  
migrated to a new Parcel format.  Something was left out.  
  
You have been given:  credvault  
  
Goal:  
1. Reverse credvault to understand the Parcel protocol.  
2. Find the vulnerability in the credential elevation path.  
3. Write solve.py (Python 3) that exploits it and reads the flag.  
4. Send solve.py here -- the server runs it against a live  
credvault instance and returns the output.  
  
Service address:  127.0.0.1 9999  (passed as argv[1] argv[2])  
Protocol:         See credvault (binary TCP, little-endian)  
  
Send your solve.py as base64.  
Paste it line by line, then send a line containing only: END  
  
[*] Waiting for solve.py (base64)...  
  
IyEvdXNyL2Jpbi9lbnYgcHl0aG9uMwppbXBvcnQgc3lzCmltcG9ydCBzb2NrZXQKaW1wb3J0IHN0cnVjdAoKZGVmIHNlbmRfY21kKHNvY2ssIGNtZCwgZGF0YT1iJycpOgogICAgbGVuZ3RoID0gbGVuKGRhdGEpCiAgICBzb2NrLnNlbmQoc3RydWN0LnBhY2soJzxJSScsIGNtZCwgbGVuZ3RoKSkKICAgIGlmIGxlbmd0aDoKICAgICAgICBzb2NrLnNlbmQoZGF0YSkKICAgIHJlc3AgPSBzb2NrLnJlY3YoOCkKICAgIGlmIGxlbihyZXNwKSAhPSA4OgogICAgICAgIHJhaXNlIFJ1bnRpbWVFcnJvcignc2hvcnQgcmVhZCBvbiByZXNwb25zZSBoZWFkZXInKQogICAgc3RhdHVzLCByZXNwX2xlbiA9IHN0cnVjdC51bnBhY2soJzxJSScsIHJlc3ApCiAgICBpZiByZXNwX2xlbjoKICAgICAgICByZXNwX2RhdGEgPSBzb2NrLnJlY3YocmVzcF9sZW4pCiAgICBlbHNlOgogICAgICAgIHJlc3BfZGF0YSA9IGInJwogICAgcmV0dXJuIHN0YXR1cywgcmVzcF9kYXRhCgpkZWYgbWFpbigpOgogICAgaWYgbGVuKHN5cy5hcmd2KSA+PSAzOgogICAgICAgIGhvc3QgPSBzeXMuYXJndlsxXQogICAgICAgIHBvcnQgPSBpbnQoc3lzLmFyZ3ZbMl0pCiAgICBlbHNlOgogICAgICAgIGhvc3QgPSAnMTI3LjAuMC4xJwogICAgICAgIHBvcnQgPSA5OTk5CgogICAgIyBCdWlsZCB0aGUgdmFsaWQgY3JlZGVudGlhbAogICAgY3JlZCA9ICgKICAgICAgICBzdHJ1Y3QucGFjaygnPEknLCAweENBMTEwMDAxKSArCiAgICAgICAgc3RydWN0LnBhY2soJzxJJywgMHhDQTExMDA0MikgKwogICAgICAgIHN0cnVjdC5wYWNrKCc8SScsIDB4MDAwMDAwMDEpICsKICAgICAgICBzdHJ1Y3QucGFjaygnPEknLCAweDAwMDAwMDAwKSArCiAgICAgICAgc3RydWN0LnBhY2soJzxJJywgMHgwMDAwMDAwMCkgKwogICAgICAgIHN0cnVjdC5wYWNrKCc8SScsIDB4MDAwMDAwNDIpCiAgICApCiAgICBpZiBsZW4oY3JlZCkgIT0gMjQ6CiAgICAgICAgcmFpc2UgUnVudGltZUVycm9yKCdjcmVkIGxlbmd0aCAhPSAyNCcpCgogICAgc29jayA9IHNvY2tldC5jcmVhdGVfY29ubmVjdGlvbigoaG9zdCwgcG9ydCkpCgogICAgc3RhdHVzLCBfID0gc2VuZF9jbWQoc29jaywgMHhjYTAwMDAwMiwgY3JlZCkKICAgIGlmIHN0YXR1cyAhPSAwOgogICAgICAgIHByaW50KGYnWyFdIFNFVCBlcnJvcjoge3N0YXR1czojeH0nLCBmaWxlPXN5cy5zdGRlcnIpCiAgICAgICAgc3lzLmV4aXQoMSkKCiAgICBzdGF0dXMsIF8gPSBzZW5kX2NtZChzb2NrLCAweGNhMDAwMDAzKQogICAgaWYgc3RhdHVzICE9IDA6CiAgICAgICAgcHJpbnQoZidbIV0gQVVUSCBlcnJvcjoge3N0YXR1czojeH0nLCBmaWxlPXN5cy5zdGRlcnIpCiAgICAgICAgc3lzLmV4aXQoMSkKCiAgICBzdGF0dXMsIF8gPSBzZW5kX2NtZChzb2NrLCAweGNhMDAwMDA0KQogICAgaWYgc3RhdHVzICE9IDA6CiAgICAgICAgcHJpbnQoZidbIV0gRUxFVkFURSBlcnJvcjoge3N0YXR1czojeH0nLCBmaWxlPXN5cy5zdGRlcnIpCiAgICAgICAgc3lzLmV4aXQoMSkKCiAgICBzdGF0dXMsIGZsYWdfZGF0YSA9IHNlbmRfY21kKHNvY2ssIDB4Y2EwMDAwMDUpCiAgICBpZiBzdGF0dXMgIT0gMDoKICAgICAgICBwcmludChmJ1shXSBGTEFHIGVycm9yOiB7c3RhdHVzOiN4fScsIGZpbGU9c3lzLnN0ZGVycikKICAgICAgICBzeXMuZXhpdCgxKQoKICAgIHByaW50KGZsYWdfZGF0YS5kZWNvZGUoKSkKCmlmIF9fbmFtZV9fID09ICdfX21haW5fXyc6CiAgICBtYWluKCkK  
END  
END  
[*] Received solve.py (1728 bytes)  
[*] Session flag generated.  
[*] Waiting for execution slot...  
[*] Slot acquired.  
[*] Starting CredVault service...  
[*] Service ready on port 9999.  
[*] Running solve.py...  
════════════════════ OUTPUT ════════════════════════  
OmniCTF{d1_02ba594d73336ce4_57d61a3a7caa8df1460c85739063f771_9f074cf8c0344e513597e327e66317f3}  
════════════════════ END OUTPUT ════════════════════  
[*] Session ended.
```
`OmniCTF{d1_02ba594d73336ce4_57d61a3a7caa8df1460c85739063f771_9f074cf8c0344e513597e327e66317f3}`
