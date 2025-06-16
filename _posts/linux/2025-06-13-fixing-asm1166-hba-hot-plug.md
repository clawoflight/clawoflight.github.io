---
title: Fixing SATA Hot Plug on an ASM1166-Based HBA
category: linux
published: true
description: "Resurrecting hot-swap functionality with alternative firmware"
comments: false
tags: ['nas', 'zfs', 'hardware']
date: 2025-06-13
---

While testing a new server build, I discovered that the SATA hot plug functionality wasn't working on my ASM1166-based HBA card. This was a critical issue as hot-swap capability is a hard requirement - without it, any drive replacement would require rebooting (and incurring downtime)

### The Problem

The card itself was functioning correctly for basic storage operations, but attempting to hot-swap any SATA device would fail. The system wouldn't recognize newly inserted drives until a full reboot, which defeated the purpose of having hot-swap capability.

### Finding a Solution

After some research, I determined this was likely a firmware issue. I found an extremely helpful [Reddit post](https://www.reddit.com/r/unRAID/comments/1j9dzxr/asm1166_firmware_mess/) that detailed similar issues with ASM1166-based controllers, which pointed me in the right direction.

This helped me find to very helpful links, which I would otherwise have looked for forever:

1. A flash tool specifically for ASM1166 controllers from Radxa's site:
   [https://dl.radxa.com/accessories/m2-to-hexa-sata-adapter/tools/](https://dl.radxa.com/accessories/m2-to-hexa-sata-adapter/tools/)

2. Firmware packages from Silverstone for their ECS06 card (which uses the same ASM1166 chip):
   [https://www.silverstonetek.com/en/product/info/expansion-cards/ECS06/](https://www.silverstonetek.com/en/product/info/expansion-cards/ECS06/)

### The Fix: Alternative Firmware

Interestingly, my card was running firmware version 221118-0048-00, which wasn't working correctly for hot-swap. The firmware from Silverstone has version 221118-0000-00 and appears to be a different build with different options enabled.

{% highlight bash %}
# Verify firmware can be read
$ sudo ./116xfwdl -S
ASM116x Firmware Update Tool V1.1.1.0

Dev[0]::FWver from PCIe: 21 11 08 00 48

# Update firmware
$ sudo ./116xfwdl -U 11080000.ROM

ASM116x Firmware Update Tool V1.1.1.0
Found 1 ASM deiceFind a SPI flash ROM ID : 68h, 40h, 13h is not in Supported List!!!Try to program...ASM116UpdateSpiFlashRom: Chip Erase status = 0
ASM116UpdateSpiFlashRom: Blank Check status = 0
ASM116UpdateSpiFlashRom: Write Data status = 0
Update SPI flash ROM......PASS!!!

# Reboot to load new firmware blob
$ sudo reboot

# Verify firmware version
$ sudo ./116xfwdl -S
ASM116x Firmware Update Tool V1.1.1.0

Dev[0]::FWver from PCIe: 21 11 08 00 00 00

{% endhighlight %}

### Testing and Results

Initial results were promising, and I was able to move one of the OS SSDs (part of a ZFS mirror) to a different SATA port without rebooting.
The new HDD I was also testing seems to be broken, so I'll need to perform more thorough testing with a different drive.
