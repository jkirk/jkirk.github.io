---
title: "Lenovo ThinkPad X1 Carbon Gen 11: Improved Debian/bookworm (netinst) Installation Journal"
create: 2023-09-28T10:26:32+0200
date: 2023-09-28T12:05:48+0200
---

My goal was to move and shrink the Windows partition, set up an encrypted partition with LVM on top of it and install Debian/bookworm.
<!--more-->

## Initial situation

The Lenovo ThinkPad X1 Carbon Gen 11 came with Windows 11.

I wanted to keep Windows 11 for rare occasions like games or firmware updates.

The partition layout looked like this:

```sh
  user@debian ~ % sudo parted /dev/nvme0n1 p
  Model: SAMSUNG MZVL21T0HDLU-00BLL (nvme)
  Disk /dev/nvme0n1: 1024GB
  Sector size (logical/physical): 512B/512B
  Partition Table: gpt
  Disk Flags:

  Number  Start   End     Size    File system  Name                          Flags
   1      1049kB  274MB   273MB   fat32        EFI system partition          boot, hidden, esp
   2      274MB   290MB   16.8MB               Microsoft reserved partition  msftres
   3      290MB   1022GB  1022GB  ntfs         Basic data partition          msftdata
   4      1022GB  1024GB  2097MB  ntfs         Basic data partition          hidden, diag
```

## Partitioning: Move and shrink Windows partition

After a lot of back and forth I decided to move the Windows partition to the end of the disk.

That way I could delete the Windows partition and expand the Debian LUKS/LVM partition easily, in case I need additional disk space.
Moving the Debian partition "to the left" once it is installed after the Windows NTFS partition would be way harder.

I fired up GParted 1.3.1 via Grml, resized the Windows NTFS partition to about 100GiB and moved it "to the right".

![](Screenshot_20230928_105755.png "GParted - Move and shrink before applying")

Applied the change:

![](Screenshot_20230928_105845.png "GParted - Applying pending operations")

![](Screenshot_20230928_110027.png "GParted - Applying pending operations II")

I was impressed: Applying the pending operations took only 1 minute and 49 seconds!

![](Screenshot_20230928_110032.png "GParted - 0 operations pending")

I rebooted the system and tested Windows boot.
To my surprise Windows booted fine.

FTR,  GParted needs to be installed manually on Debian Live.

## Partitioning: Prepare Debian partitions

> ℹ️ **Note**: The following partitioning can and most probably should be accomplished with the Debian Installer.

Created the Debian partitions (boot and encrypted LVM):

```sh
  root@grml ~ # parted /dev/nvme0n1
  GNU Parted 3.6
  Using /dev/nvme0n1
  Welcome to GNU Parted! Type 'help' to view a list of commands.
  (parted) p
  Model: SAMSUNG MZVL21T0HDLU-00BLL (nvme)
  Disk /dev/nvme0n1: 1024GB
  Sector size (logical/physical): 512B/512B
  Partition Table: gpt
  Disk Flags:

  Number  Start   End     Size    File system  Name                          Flags
   1      1049kB  274MB   273MB   fat32        EFI system partition          boot, hidden, esp, no_automount
   2      274MB   290MB   16.8MB               Microsoft reserved partition  msftres
   3      918GB   1022GB  104GB   ntfs         Basic data partition          msftdata
   4      1022GB  1024GB  2097MB  ntfs         Basic data partition          hidden, diag, no_automount

  (parted) mkpart primary ext4 290MB 2290MB
  (parted) mkpart primary ext4 2290MB 918GB
  (parted) p
  Model: SAMSUNG MZVL21T0HDLU-00BLL (nvme)
  Disk /dev/nvme0n1: 1024GB
  Sector size (logical/physical): 512B/512B
  Partition Table: gpt
  Disk Flags:

  Number  Start   End     Size    File system  Name                          Flags
   1      1049kB  274MB   273MB   fat32        EFI system partition          boot, hidden, esp, no_automount
   2      274MB   290MB   16.8MB               Microsoft reserved partition  msftres
   5      290MB   2290MB  2000MB  ext4         primary
   6      2290MB  918GB   916GB   ext4         primary
   3      918GB   1022GB  104GB   ntfs         Basic data partition          msftdata
   4      1022GB  1024GB  2097MB  ntfs         Basic data partition          hidden, diag, no_automount

  (parted) q
  Information: You may need to update /etc/fstab.
```

Initialized the LUKS partition:

```sh
  root@grml ~ # cryptsetup luksFormat /dev/nvme0n1p6

  WARNING!
  ========
  This will overwrite data on /dev/nvme0n1p6 irrevocably.

  Are you sure? (Type 'yes' in capital letters): YES
  Enter passphrase for /dev/nvme0n1p6:
  Verify passphrase:
  cryptsetup luksFormat /dev/nvme0n1p6  18.68s user 0.75s system 117% cpu 16.481 total
```

Unlocked the LUKS partition and set up LVM:

```sh
  root@grml ~ # cryptsetup open /dev/nvme0n1p6 nvme0n1p6_crypt
  Enter passphrase for /dev/nvme0n1p6:
  cryptsetup open /dev/nvme0n1p6 nvme0n1p6_crypt  5.37s user 0.21s system 56% cpu 9.862 total
  root@grml ~ # pvcreate /dev/mapper/nvme0n1p6_crypt
    Physical volume "/dev/mapper/nvme0n1p6_crypt" successfully created.
  root@grml ~ # vgcreate vg0-predator /dev/mapper/nvme0n1p6_crypt
    Volume group "vg0-predator" successfully created
  root@grml ~ # lvcreate -L 16GB -n rootfs vg0-predator
    Logical volume "rootfs" created.
  root@grml ~ # sudo lvcreate -L 50GB -n home vg0-predator
    Logical volume "home" created.
```

> ℹ️ **Note**: I should have created a 32GB swap LV at this point but missed to do so.

## Install Debian/bookworm with Cinnamon

Booted `debian-12.1.0-amd64-netinst.iso` via Ventoy and started the Graphical Debian-Installer.

* Language: English
* Location: Europe/Austria
* Locales: en_US.UTF-8 (Note: en_DK.UTF-8 was not listed)
* Keyboard: American English (Note: US, intl. with dead keys was not listed)
* Network: Unknown interface (Note: I used my USB-C LAN network adapter)
* Hostname: predator
* Domain name: home.syn-net.org
* Root password: (empty) (Note: If empty, sudo will be configured for the initial user)
* Full name: Darshaka Pathirana
* Username: jkirk
* Set password for user jkirk
* Partition disks: manual
* Partition disk:

(screenshots)

* Started a shell and run

```sh
  ~ # cryptsetup open /dev/nvme0n1p6 nvme0n1p6_crypt
  ~ # pvscan
  ~ # vgchange -ay
```

* Partition disk: selected rootfs and home

* Finished partitioning and wrote changes to disk

  Swap was recommended (I will create 32GB swap LV later)

After that the base installation was performed.

* Mirror country: Austria / debian.anexia.at (no proxy)
* Participate in package usage survey: yes
* Software selection:

  * Debian desktop environment
  * Cinnamon
  * SSH server
  * standard system utilities

After the installation and the reboot, Debian dropped into initramfs shell, with an ALERT that rootfs does not exist:

I had to unlock partition manually (`cryptsetup open`), because `/etc/crypttab` was not set up on the installation.

I then pressed Ctrl-d to exit the shell and the system booted fine.

After booting Debian the system looked like this:

```
  jkirk@predator:~$ lsblk -o +LABEL,FSTYPE,UUID
  NAME                       MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS LABEL     FSTYPE      UUID
  sda                          8:0    1 119.5G  0 disk
  ├─sda1                       8:1    1 119.5G  0 part              Ventoy    exfat       F966-2D8E
  └─sda2                       8:2    1    32M  0 part              VTOYEFI   vfat        7CDB-00D9
  nvme0n1                    259:0    0 953.9G  0 disk
  ├─nvme0n1p1                259:1    0   260M  0 part  /boot/efi   SYSTEM    vfat        50D7-B9E5
  ├─nvme0n1p2                259:2    0    16M  0 part
  ├─nvme0n1p3                259:3    0  97.1G  0 part              Windows   ntfs        46B6DA09B6D9F8FF
  ├─nvme0n1p4                259:4    0     2G  0 part              WinRE_DRV ntfs        DE1EDA651EDA366D
  ├─nvme0n1p5                259:5    0   1.9G  0 part  /boot                 ext4        4b42a4ed-14f5-42c4-b006-9f1ceca7ad11
  └─nvme0n1p6                259:6    0 852.7G  0 part                        crypto_LUKS ff92d04d-b964-4ff2-89c7-11e628a4b7fa
    └─nvme0n1p6_crypt        254:0    0 852.7G  0 crypt                       LVM2_member Lve3cl-81fM-XX0z-K5rm-uq4f-DBU0-3wtyUJ
      ├─vg0--predator-rootfs 254:1    0    16G  0 lvm   /                     ext4        68e7372c-ee62-4172-97c1-f71f9a2bfaaa
      └─vg0--predator-home   254:2    0    50G  0 lvm   /home                 ext4        7cdee128-ae26-48e6-af27-89ee98b984f9

Fixed the `/etc/crypptab` problem:

```sh
  jkirk@predator:~$ sudo vi /etc/crypttab
  jkirk@predator:~$ cat /etc/crypttab
  # <target name>	<source device>		<key file>	<options>
  nvme0n1p6_crypt	UUID=ff92d04d-b964-4ff2-89c7-11e628a4b7fa	none	luks

  jkirk@predator:~$ sudo update-initramfs -u -k all
  update-initramfs: Generating /boot/initrd.img-6.1.0-12-amd64
  update-initramfs: Generating /boot/initrd.img-6.1.0-10-amd64
```
