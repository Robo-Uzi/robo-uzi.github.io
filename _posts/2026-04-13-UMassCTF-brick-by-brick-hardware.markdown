---
layout: post
title:  "Brick by Brick"
date:   2026-04-13 18:52:00 -0400
author: robo.uzi
tags: [CTF]
permalink: /UMassCTF-2026-brick-by-brick-hardware/
---

**Category**: hardware medium

**Author**: vdm_0x (Vadim)

**Description**: We intercepted a signal from the Master Builder's workshop. Looks like someone was communicating with a custom LEGO controller, but all we captured was this raw data. Can you piece together the message?

I get the challenge file:
```shell
unzip brick-by-brick.zip  
Archive:  brick-by-brick.zip  
  inflating: code.csv
  
cat code.csv | wc -l  
56991
  
cat code.csv | head  
timestamp,logic_level  
0.0,1  
2.9960751415645503e-05,1  
5.992150283129101e-05,1  
8.988225424693651e-05,1  
0.00011984300566258201,1  
0.00014980375707822753,0  
0.00017976450849387305,1  
0.00020972525990951856,1  
0.00023968601132516408,0  

cat code.csv | tail  
1.7071636156613534,0  
1.707193576412769,1  
1.7072235371641846,0  
1.7072534979156002,0  
1.7072834586670158,1  
1.7073134194184314,1  
1.707343380169847,1  
1.7073733409212626,1  
1.7074033016726782,1  
1.7074332624240938,1
```

UART transmits data as a sequence of voltage levels over time:
- `1` = high voltage (idle state)
- `0` = low voltage

The CSV is just a sampled digital waveform of the signal over time. 

UART framing (how the message is structured):
```shell
Idle: 1 (line stays high)  
Start: 0  
Data: 8 bits (LSB first)  
Stop: 1
```

Solve script:
```python
import csv

bits = []
with open('code.csv', 'r') as f:
    reader = csv.reader(f)
    next(reader)  # skip header
    for row in reader:
        bits.append(int(row[1]))

i = 0
result = []
while i < len(bits):
    # skip idle high bits
    while i < len(bits) and bits[i] == 1:
        i += 1
    if i >= len(bits):
        break
    # start bit (0) consume it
    i += 1
    # read 8 data bits (LSB first)
    if i + 8 > len(bits):
        break
    data = 0
    for j in range(8):
        data |= (bits[i + j] << j)
    i += 8
    # stop bit should be 1; consume it
    if i < len(bits) and bits[i] == 1:
        i += 1
    result.append(chr(data))

print(''.join(result))
```

Output:
```shell
python3 solve4.py  
[    0.000000] Linux version 6.8.1-arch1-1 (builder@archlinux) (gcc (GCC) 13.2.1) #1 SMP PREEMPT_DYNAMIC  
	[    0.000000] Command line: BOOT_IMAGE=/vmlinuz-linux root=UUID=abcd-1234 rw quiet  
	[    0.000000] BIOS-provided physical RAM map:  
	[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x000000000009ffff] usable 
	[    0.000000] BIOS-e820: [mem 0x0000000000100000-0x000000003fffffff] usable 
	[    0.000000] NX (Execute Disable) protection: active  
	[    0.000000] efi: EFI v2.8 by American Megatrends  
	[    0.000000] efi: ACPI=0x3ffe0000 ACPI 2.0=0x3ffe0000 SMBIOS=0x3ffb0000  
	[    0.000000] SMBIOS 3.3.0 present.  
	[    0.000000] DMI: ASUS System Product Name, BIOS 2401 06/15/2024  
	[    0.000000] tsc: Detected 3600.000 MHz processor  
	[    0.000001] e820: update [mem 0x00000000-0x00000fff] usable ==> reserved  
	[    0.000002] last_pfn = 0x40000 max_arch_pfn = 0x400000000  
	[    0.000010] x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- WT  
	[    0.000100] found SMP MP-table at [mem 0x000f5a40-0x000f5a4f]  
	[    0.000200] ACPI: Early table checksum verification disabled  
	[    0.000300] ACPI: RSDP 0x000000003FFE0000 000024 (v02 AMI)  
	[    0.000400] ACPI: XSDT 0x000000003FFE1000 0000F4 (v01 AMI)  
	[    0.000500] ACPI: FACP 0x000000003FFD0000 000114 (v06 AMI)  
	[    0.000600] ACPI: DSDT 0x000000003FFC0000 00C123 (v02 AMI)  
	[    0.000700] ACPI: APIC 0x000000003FFD1000 000084 (v04 AMI)  
	[    0.001000] CPU0: Intel(R) Core(TM) i7-9700K CPU @ 3.60GHz  
	[    0.001100] CPU0: APIC id 0, version 0x14  
	[    0.002000] smpboot: CPU0: Intel(R) Core(TM) i7-9700K (family: 0x6, model: 0x9e)  
	[    0.002100] Performance Events: PEBS fmt3+, Skylake events  
	[    0.003000] smp: Bringing up secondary CPUs ...  
	[    0.010000] smp: Brought up 8 nodes, 8 CPUs  
	[    0.011000] devtmpfs: initialized  
	[    0.012000] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff 
	[    0.013000] futex hash table entries: 2048  
	[    0.014000] NET: Registered protocol family 16  
	[    0.015000] audit: initializing netlink subsys (disabled)  
	[    0.016000] PCI: Using configuration type 1 for base access  
	[    0.017000] PCI: Probing PCI hardware  
	[    0.018000] pci 0000:00:1f.3: Intel HD Audio controller  
	[    0.019000] usbcore: registered new interface driver usbfs  
	[    0.020000] usbcore: registered new interface driver hub  
	[    0.021000] usbcore: registered new device driver usb  
	[    0.022000] SCSI subsystem initialized  
	[    0.023000] libata version 3.00 loaded  
	[    0.024000] ahci 0000:00:17.0: version 3.0  
	[    0.025000] scsi host0: ahci  
	[    0.026000] ata1: SATA max UDMA/133 abar m2048@0xf7a00000 port 0xf7a00100 irq 126  
	[    0.030000] ata1: SATA link up 6.0 Gbps (SStatus 133 SControl 300)  
	[    0.031000] ata1.00: ATA-10: Samsung SSD 970 EVO 1TB, 2B2QEXE7  
	[    0.032000] ata1.00: 1953525168 sectors, multi 1: LBA48 NCQ (depth 32)  
	[    0.033000] scsi 0:0:0:0: Direct-Access     ATA Samsung SSD 970 1B2Q PQ: 0 ANSI: 5  
	[    0.034000] sd 0:0:0:0: [sda] 1953525168 512-byte logical blocks  
	[    0.035000] sd 0:0:0:0: [sda] Write Protect is off  
	[    0.036000] EXT4-fs (sda2): mounted filesystem with ordered data mode  
	[    0.037000] systemd[1]: systemd 255 running in system mode  
	[    0.037500] secretflag: 554d4153537b553452375f31355f3768335f623335372c5f72316768373f7d  
	[    0.038000] systemd[1]: Detected architecture x86-64.  
	[    0.039000] systemd[1]: Hostname set to <archlinux>.  
	[    0.040000] systemd[1]: Reached target Basic System.  
	[    0.041000] systemd[1]: Started Network Manager.  
	[    0.042000] systemd[1]: Started Login Service.  
	[    0.043000] systemd[1]: Reached target Graphical Interface.
```
The flag is in hex. 

`UMASS{U4R7_15_7h3_b357,_r1gh7?}`
