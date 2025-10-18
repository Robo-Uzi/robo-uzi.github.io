---
layout: post
title:  "SmartCoffee"
date:   2025-10-18 16:16:43 -0400
author: robo.uzi
tags: [ctf]
permalink: /qnqsec-2025-smartcoffee/
---

**Title:** SmartCoffee

**Category:** hardware

**Description:** A firmware update introduced unexpected behavior to a SmartCoffee device. Investigate the firmware image for a hidden secret.

**Author:** Azadin

I get two files. `firmware_x86.elf` and `logs.txt`:
```shell
file firmware_x86.elf  
firmware_x86.elf: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=20de7fc4732ba9cd132a125b037dfb36c1e38c3c, for GNU/Linux 3.2.0, stripped

cat logs.txt  
Booting SmartCoffee v1.2 (build 2025-10-15)  
[    0.000000] Early console initialized  
[    0.004321] CPU: ARM Cortex-M4 r0p1  
[    0.012348] rtc-pcf8563: registered as rtc0  
[    0.045901] mmc0: host init - card present  
[    0.120014] i2c /dev/i2c-1: detected device at 0x50  
[    0.120256] i2c /dev/i2c-1: detected device at 0x1a  
[    0.135602] spi0.0: registered flash (winbond)  
[    0.220731] adc: calibration complete  
[    0.332100] gpio: irqchip registered  
[    0.995012] eeprom: reading primary image (addr=0x0000, len=8192)  
[    0.995030] eeprom: checksum 0x4f2a  
[    1.120512] usbcore: new interface driver usb-storage  
[    1.221003] wlan0: firmware missing - disabled  
[    1.332445] system: services startup complete  
[    1.500000] watchdog: feeding watchdog  
--- device info ---  
Device: SmartCoffee v1.2  
PCB: SC-2025-REV1  
Device Serial: SC-01-ABC123  
Bootloader: U-Boot 2021.04  
Kernel: 5.10.124  
[    2.000000] entering userland  
[    2.100000] diagnostics: self-test passed
```

Putting `firmware_x86.elf` in ghidra I find this function:
```d
void FUN_001011f0(void)

{
  long lVar1;
  
  lVar1 = 0;
  puts("EEPROM DUMP (raw):");
  do {
    while( true ) {
      printf("%02x ",(ulong)(byte)(&DAT_001020e0)[lVar1]);
      if ((~(uint)lVar1 & 7) != 0) break;
      lVar1 = lVar1 + 1;
      putc(10,stdout);
      if (lVar1 == 0x21) goto LAB_00101254;
    }
    lVar1 = lVar1 + 1;
  } while (lVar1 != 0x21);
LAB_00101254:
  putc(10,stdout);
  puts("(Note: bytes are likely obfuscated.)");
  return;
}
```

So `logs.txt` shows the device prints an EEPROM dump at boot (An EEPROM dump is a snapshot of all the data stored in an EEPROM (Electrically Erasable Programmable Read-Only Memory) of a device). In ghidra the firmware prints 33 bytes from `DAT_001020e0`:
```d
// prints 0x21 (33) bytes from DAT_001020e0: "EEPROM DUMP (raw):"
printf("%02x ",(ulong)(byte)(&DAT_001020e0)[lVar1]);
```

I find the offset I need like this: `file_offset = 0x001020e0 - 0x00100000 = 0x20e0`

`0xC4` was found by testing single byte XOR keys across 0-255 on the 33 bytes and looking for readable output:
```shell
dd if=firmware_x86.elf bs=1 skip=$((0x20e0)) count=33 2>/dev/null | python3 -c "import sys; print(sys.stdin.buffer.read().translate(bytes([i^0xC4 for i in range(256)]) ).decode())"
QnQSec{3x_s3snum_c4n_d0_h4rdw4r3}
```
This extracts the 33 bytes at offset `0x20e0`, XORs each byte with `0xC4`, then prints the resulting text.

`QnQSec{3x_s3snum_c4n_d0_h4rdw4r3}`