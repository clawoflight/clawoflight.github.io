---
title: Migrating an Ubuntu Server installation from BIOS to UEFI (Part 1)
category: announcements
published: true
comments: false
date: 2019-10-23
---

I recently tried rebooting my NAS into a thumb drive with a custom Arch Linux maintenance image (containing the ZFS kernel modules and directly accessible over SSH).
After a quick Google, I found that `efibootmgr` should be capable of doing that in one command --- until I realized that I was booted in BIOS mode, not UEFI.
Some investigation later, I think the reason is that I used a non-UEFI-compatible GPU for the installation, which caused the Mainboard to boot in CSM mode.

Of course, nice features like choosing what to reboot into do not work in BIOS compatibility mode, so I need to get that sorted out.
I do not have access to an unused UEFI-compatible GPU at the moment, but I figured I would do what I could until then.

I found a [guide](https://serverfault.com/questions/963178/how-do-i-convert-my-linux-disk-from-mbr-to-gpt-with-uefi) on serverfault which I was able to adapt for my needs.
Since I was already using the GPT partitioning scheme, I did not need to convert from MBR and could skip the first 5 steps.

### Freeing up space

I needed to make space for the EFI System Partition (ESP).
Thankfully, I use LVM on root and had a lot of breathing room, so I did not need to resize the root FS or even the LV or VG, but only the PV (LVM physical volume).

{% highlight bash %}
    sudo pvresize --setphysicalvolumesize 230G /dev/sdd3
{% endhighlight %}

Just to be sure that a reboot would work, because I have little experience with LVM, I also checked a few things:

* Do the UUIDs still match the values in the fstab? (`sudo blkid /dev/mapper/ubuntu--vg-ubuntu--lv /boot && cat /etc/fstab`)

* Do the LV and VG UUIDs still match the values passed to the kernel? (`sudo vgdisplay && sudo lvdisplay && cat /boot/grub/grub.cfg`)
  where grub.cfg contains 'lvmid/VGUUID/LVUUID' multiple times.

  {% highlight %}\
  set root='lvmid/dFkRYx-xI3s-oKSh-E2dA-DRma-oNwm-fuoe9W/AF86OL-Ginq-fbrD-fdoh-slPc-NSA1-isgdSN'
  if [ x$feature*platform*search_hint = xy ]; then
  search --no-floppy --fs-uuid --set=root --hint='lvmid/dFkRYx-xI3s-oKSh-E2dA-DRma-oNwm-fuoe9W/AF86OL-Ginq-fbrD-fdoh-slPc-NSA1-isgdSN
  ' e2281bec-de5f-406b-815e-cb81b2c80619
  {% endhighlight %}

  If this hadn't matched, I would have run `sudo update-grub`and hoped for the best.
  But apparently LVM UUIDs are independent of partition UUIDs, so I did not run into any problems with this at any step, even after resizing the physical partition.

### Creating an ESP

  Now, the next step is to resize the partition containing the LVM PV. To be sure I don't overwrite everything, I left some buffer between the partitions. And just to remove any unit conversion ambiguity, I calculated everything in sector sizes.

  {% highlight %}
  sudo pvs --units s
  [sudo] password for bennett:
  PV VG Fmt Attr PSize PFree
  /dev/sdd3 ubuntu-vg lvm2 a-- 482459648S 241287168S
  {% endhighlight %}

  `PSize` here is the size of the partition in sectors. The physical partition size can be viewed with `parted /dev/sdd unit s p`.

  Partitions cannot be resized, so the LVM partition needs to be deleted and re-created smaller.
  Be sure to use the sector size shown in `pvs` as the new partition size.

  Now we can create the ESP, which I did using `gdisk` to easily set the label (to "ESP" because I am very creative). Let's update the kernel's view of the drive with `sudo partprobe /dev/sdd`. The final step is to create the file system - UEFI expects FAT32/VFAT: `sudo mkfs -t vfat -v /dev/disk/by-partlabel/ESP`

  The final step is to mount the partition:

  {% highlight %}
  sudo mkdir /boot/efi
  echo /dev/disk/by-partlabel/ESP /boot/efi vfat defaults 0 2 | sudo tee -a /etc/fstab
  sudo mount /boot/efi
  {% endhighlight %}

### Next Step: re-installing GRUB for UEFI

I need to get another UEFI-capable GPU first. Then, I think it boils down to a few steps:

- verify the settings in UEFI setup
- install GRUB in EFI-mode. [these](https://askubuntu.com/questions/509423/which-commands-to-convert-a-ubuntu-bios-install-to-efi-uefi-without-boot-repair) [pages](https://wiki.archlinux.org/index.php/GRUB#UEFI_systems) might help.
- manually boot from that disk using UEFI
- verify that I am booted in UEFI and re-install GRUB to set it as default boot loader.
- try rebooting into a thumb using `efibootmgr -n`.

