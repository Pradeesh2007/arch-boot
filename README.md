# Arch Linux Manual Installation Guide (Without `archinstall`)

> A comprehensive guide to manually installing Arch Linux while understanding **what each step does**, **why it is necessary**, and **how Linux works internally**.
>
> This guide is intended for students, Linux enthusiasts, system administrators, cloud engineers, cybersecurity learners, and anyone who wants to understand Arch Linux beyond simply following commands.

---

# Table of Contents

1. Introduction
2. Installation Philosophy
3. System Requirements
4. Boot Process Overview
5. Phase 1 – Boot the Live Environment
6. Phase 2 – Connect to the Internet
7. Phase 3 – Identify Storage Devices
8. Phase 4 – Partition the Disk
9. Phase 5 – Create Filesystems
10. Phase 6 – Mount the Filesystems
11. Phase 7 – Install the Base System
12. Phase 8 – Generate the Filesystem Table
13. Phase 9 – Change Root (chroot)
14. Phase 10 – Configure the Installed System
15. Phase 11 – Install the Bootloader
16. Phase 12 – Finish Installation
17. Boot Sequence Explained
18. Important Linux Concepts
19. Common Mistakes
20. Troubleshooting
21. References

---

# Introduction

Unlike many Linux distributions, Arch Linux does not hide the installation process behind a graphical installer.

Instead, it allows you to build your operating system from the ground up.

This manual installation process teaches:

- Linux filesystem hierarchy
- Disk partitioning
- Filesystems
- Mounting
- Package management
- Boot process
- System configuration
- Bootloaders
- UEFI
- Linux kernel basics

By completing this installation manually, you gain knowledge that applies to nearly every Linux distribution.

---

# Installation Philosophy

The installation can be viewed as constructing a house.

| House | Linux Installation |
|---------|-------------------|
| Buy land | Boot Live ISO |
| Divide land | Partition Disk |
| Build rooms | Create Filesystems |
| Connect rooms | Mount Filesystems |
| Build house | Install Base System |
| Enter house | chroot |
| Install electricity | Configure System |
| Install front door | Install Bootloader |
| Move in | Reboot |

---

# System Requirements

Minimum:

- 64-bit CPU
- 2 GB RAM
- 20 GB Storage
- Internet Connection
- USB Drive
- UEFI Firmware (recommended)

---

# Boot Process Overview

```
Power On
      │
      ▼
UEFI Firmware
      │
      ▼
EFI System Partition
      │
      ▼
GRUB Bootloader
      │
      ▼
Linux Kernel
      │
      ▼
systemd
      │
      ▼
Mount Filesystems
      │
      ▼
Login Prompt
```

---

# Phase 1 — Boot the Live Environment

## Objective

Start the temporary Arch Linux operating system from the USB.

The Arch ISO runs entirely from RAM.

Nothing is installed on your SSD yet.

### Verify Boot Mode

```bash
ls /sys/firmware/efi
```

If the directory exists, you booted using UEFI.

---

# Phase 2 — Connect to the Internet

The installer downloads packages from official Arch repositories.

Without Internet, installation cannot continue.

Ethernet:

```bash
ping archlinux.org
```

WiFi:

```bash
iwctl
```

Example:

```
device list

station wlan0 scan

station wlan0 get-networks

station wlan0 connect YOUR_WIFI
```

Exit:

```
exit
```

---

# Phase 3 — Identify Storage Devices

Display disks:

```bash
lsblk
```

Example:

```
sda
├── sda1
└── sda2

nvme0n1
├── nvme0n1p1
└── nvme0n1p2
```

Linux treats storage devices as files under:

```
/dev/
```

Examples:

```
/dev/sda

/dev/sdb

/dev/nvme0n1
```

---

# Phase 4 — Partition the Disk

## Purpose

A partition divides one physical disk into logical sections.

Recommended Layout

| Partition | Purpose |
|------------|----------|
| EFI | Bootloader |
| Root | Operating System |

Create GPT:

```bash
fdisk /dev/sda
```

Commands:

```
g
n
t
w
```

---

# Phase 5 — Create Filesystems

A partition is empty storage.

A filesystem organizes data.

Without a filesystem, Linux cannot store files.

EFI:

```bash
mkfs.fat -F32 /dev/sda1
```

Root:

```bash
mkfs.ext4 /dev/sda2
```

---

# Phase 6 — Mount Filesystems

Linux uses one unified directory tree.

Mount Root:

```bash
mount /dev/sda2 /mnt
```

Mount EFI:

```bash
mkdir /mnt/boot

mount /dev/sda1 /mnt/boot
```

Now `/mnt` becomes the future root filesystem.

---

# Phase 7 — Install the Base System

Install essential packages.

```bash
pacstrap /mnt base linux linux-firmware vim
```

Package Purpose

| Package | Purpose |
|----------|----------|
| base | Essential Linux utilities |
| linux | Linux Kernel |
| linux-firmware | Hardware firmware |
| vim | Text editor |

---

# Phase 8 — Generate fstab

Generate automatic mount configuration.

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

This file tells Linux:

- what to mount
- where to mount
- how to mount

during every boot.

---

# Phase 9 — chroot

Enter the installed system.

```bash
arch-chroot /mnt
```

Before:

```
/
```

means

```
Live ISO
```

After:

```
/
```

means

```
Installed System
```

---

# Phase 10 — Configure the Installed System

## Timezone

```bash
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime

hwclock --systohc
```

---

## Locale

Edit:

```
/etc/locale.gen
```

Enable:

```
en_US.UTF-8 UTF-8
```

Generate:

```bash
locale-gen
```

Create locale:

```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

---

## Hostname

```bash
echo "archlinux" > /etc/hostname
```

---

## Hosts

```
127.0.0.1 localhost
::1 localhost
127.0.1.1 archlinux.localdomain archlinux
```

---

## Root Password

```bash
passwd
```

---

# Phase 11 — Install Bootloader

Install packages:

```bash
pacman -S grub efibootmgr
```

Install GRUB:

```bash
grub-install \
--target=x86_64-efi \
--efi-directory=/boot \
--bootloader-id=GRUB
```

Generate configuration:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

---

# Phase 12 — Finish Installation

Exit:

```bash
exit
```

Unmount:

```bash
umount -R /mnt
```

Reboot:

```bash
reboot
```

Remove the installation USB.

---

# Boot Sequence Explained

```
Power Button
      │
      ▼
Motherboard Firmware
      │
      ▼
UEFI
      │
      ▼
EFI Partition
      │
      ▼
GRUB
      │
      ▼
Linux Kernel
      │
      ▼
Init (systemd)
      │
      ▼
Mount Root Filesystem
      │
      ▼
Start Services
      │
      ▼
Display Login Screen
```

---

# Important Linux Concepts

## Filesystem

Organizes files on storage.

Examples:

- ext4
- FAT32
- XFS
- Btrfs

---

## Mounting

Linux combines every storage device into one directory tree.

Everything begins at:

```
/
```

---

## Kernel

The core of Linux.

Responsibilities:

- CPU Scheduling
- Memory Management
- Device Drivers
- Process Management
- Filesystems

---

## Bootloader

Loads the Linux kernel.

Examples:

- GRUB
- systemd-boot
- rEFInd

---

## chroot

Temporarily changes the root directory.

Useful for:

- Installation
- Recovery
- Rescue Systems

---

# Common Mistakes

- Forgetting to mount the EFI partition
- Installing GRUB to the wrong location
- Incorrect disk selection
- Missing Internet connection
- Forgetting to generate `fstab`
- Skipping locale configuration
- Not setting a root password

---

# Troubleshooting

## No Boot Device Found

Possible causes:

- GRUB not installed
- Wrong EFI partition
- Incorrect UEFI settings

---

## Cannot Connect to WiFi

Verify:

```
iwctl
```

Check interface:

```
device list
```

---

## pacstrap Failed

Usually:

- No Internet
- Mirror issue

---

# References

- Arch Linux Wiki
- Arch Installation Guide
- Linux Filesystem Hierarchy Standard (FHS)
- GNU GRUB Documentation

---

# License

MIT License

---

# Author

Created as a personal learning project to understand Linux from the ground up and to document the manual Arch Linux installation process for future reference.
