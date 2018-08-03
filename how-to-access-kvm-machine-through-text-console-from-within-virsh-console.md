---
title: how to access kvm machine through text console from within virsh console
date: 2018-05-07 17:06:49
tags: [kvm,console,serial,port]
categories: virtualization
---

When kernel crashes,  stack cannot be dumped through ssh connection.
It's a better way to add a serial port to our virtual machine.
When it crashes, some import information, such as `printk` will be printed from the serial port, and you can easily obtain these output from host.

## For older grub
### Steps 1.
`edit /boot/grub/menu.lst`  or `edit /boot/grub/grub.cfg`
append the following to the end to the boot menu
```
console=ttyS0,115200
```
### Step 2.
on host machine
```bash
virsh console [your-vm-domain]
```

## For latest grub
```bash
sudo vi /etc/default/grub

GRUB_CMDLINE_LINUX_DEFAULT=”console=ttyS0″

sudo update-grub2
```
