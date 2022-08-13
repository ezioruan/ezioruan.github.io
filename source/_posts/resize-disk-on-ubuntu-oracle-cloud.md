---
title: resize disk sise for ubuntu instance in Oracle Cloud
date: 2022-02-08 8:18:15
tags: tools
---

oracle has tools to increase disk sise for Oracle Image, but not ubuntu, we need to do it manualy


The script internally calls growpart from cloud-guest-utils and resize2fs. So if you are not using LVM etc, to grow main ext4 partition of a boot volume in Ubuntu, simply run:

```
sudo growpart /dev/sda 1
sudo resize2fs -z ./sda1.e2undo /dev/sda1
```

Since Oracle puts the boot partition after main, to be safe you can also install efibootmgr and check 
```
efibootmgr -v 
```
output. If yours also looks like

```
Boot0002* UEFI ORACLE BlockVolume       PciRoot(0x0)/Pci(0x12,0x7)/Pci(0x0,0x0)/SCSI(0,1)N.....YM....R,Y.
```
then it means it uses SCSI disk/partition number to locate boot partition and it should be safe to reboot now.
