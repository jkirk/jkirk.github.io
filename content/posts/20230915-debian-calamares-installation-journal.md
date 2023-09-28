---
title: "Lenovo ThinkPad X1 Carbon Gen 11: Debian/bookworm Calamares Installation Journal"
create: 2023-09-15T00:11:06Z
date: 2023-09-28T10:01:51+0200
---

tl;dr: Avoid the GUI Installer (Calamares) of Debian Live
<!--more-->

## Warning

I booted Debian Live 12.1.0 (bookworm) / Cinnamon and tried the Calamares Installer.

My goal was to resize the Windows partition and set up encrypted partition with LVM on top of it.
This it absolutely did not work for me! 🔥

TODO: A really do not recommend using the installer, there are too many bugs, see: https://bugs.debian.org/cgi-bin/pkgreport.cgi?pkg=calamares;dist=unstable

## Partitioning

The Lenovo ThinkPad X1 Carbon Gen 11 comes with Windows 11.

I wanted to keep Windows 11 for rare occasions like games or firmware updates.

The system was partitioned like this:

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

## Calamares installer for Debian 12 (bookworm) - Install alongside

![](Screenshot_20230831_192815.png "Welcome to the Calamares installer for Debian 12 (bookworm)")

![](Screenshot_20230831_213327.png "Location")

![](Screenshot_20230831_213339.png "Keyboard")

![Install alongside](Screenshot_20230927_222247.png "Partitions")

![](Screenshot_20230831_204359.png "Users")

![](Screenshot_20230927_224715.png "All done.")

After the reboot, Debian booted fine.
The partition looked like this, Debian was installed into a large ext4 partition:

```sh
  Model: SAMSUNG MZVL21T0HDLU-00BLL (nvme)
  Disk /dev/nvme0n1: 1024GB
  Sector size (logical/physical): 512B/512B
  Partition Table: gpt
  Disk Flags:

  Number  Start   End     Size    File system  Name                          Flags
   1      1049kB  274MB   273MB   fat32        EFI system partition          boot, hidden, esp
   2      274MB   290MB   16,8MB               Microsoft reserved partition  msftres
   3      290MB   103GB   103GB   ntfs         Basic data partition          msftdata
   5      103GB   1022GB  919GB   ext4         root
   4      1022GB  1024GB  2097MB  ntfs         Basic data partition          hidden, diag
```

I was expecting a few more options when I selected 'Install alongside', but ok...

## Calamares installer for Debian 12 (bookworm) - Manual Partitioning

As "Install Alongside" did not give me the layout I wanted, I [reverted](#how-to-revert-the-partition-layout) the partition layout and tried "Manual Partitioning".

![](Screenshot_20230927_233835.png "Partitions - Manual partitioning")

I selected the Windows NTFS partition and resized the partition to about 100GiB:

![](Screenshot_20230927_233902.png "Partitions - Manual partitioning II")

![](Screenshot_20230927_233928.png "Partitions - Edit Existing (NTFS) Partition")

There was now enough free space to create two more partitions.

First, I created a boot partition with 2000 MiB:

![](Screenshot_20230927_233958.png "Partitions - Manual partitioning III")

![](Screenshot_20230927_234143.png "Partitions - Create a (ext4, boot) Partition")

As I wanted an encrypted LVM partition I first tried to create LUKS partition:

![](Screenshot_20230927_234415.png "Partitions - Create a (luks2) Partition")

But selecting "lvm2 pv" with the "encrypt" option seemd to make more sense.
I did not set the 'lvm' flag (as the partition itself is LUKS).

![](Screenshot_20230927_234608.png "Partitions - Create a (encypted lvm2 pv) Partition")

The boot and the LUKS partition were not applied at that point, it is unclear how to apply the change:

![](Screenshot_20230831_214201.png "Partition - Manual partitioning IV")

I selected "New Volume Group" on this "New partition" and set the Volume Group Name `vg0-predator`:

![](Screenshot_20230927_234637.png "New volume group")

After clicking OK, everything was lost:

![](Screenshot_20230927_234659.png "Partitions - Manual partition V")

So, this also did not work.

## Calamares installer for Debian 12 (bookworm) - Command Line Partitioning

Before running the installer I decided to manually partition the disk with the good old command line tools.

I first resized the NTFS partition:

```sh
  user@debian ~ % sudo ntfsresize -s 100000M /dev/nvme0n1p3
  ntfsresize v2022.10.3 (libntfs-3g)
  Device name        : /dev/nvme0n1p3
  NTFS volume version: 3.1
  Cluster size       : 4096 bytes
  Current volume size: 1021821579776 bytes (1021822 MB)
  Current device size: 1021821583360 bytes (1021822 MB)
  New volume size    : 99999998464 bytes (100000 MB)
  Checking filesystem consistency ...
  100.00 percent completed
  Accounting clusters ...
  Space in use       : 56667 MB (5.5%)
  Collecting resizing constraints ...
  Needed relocations : 0 (0 MB)
  WARNING: Every sanity check passed and only the dangerous operations left.
  Make sure that important data has been backed up! Power outage or computer
  crash may result major data loss!
  Are you sure you want to proceed (y/[n])? y
  Schedule chkdsk for NTFS consistency check at Windows boot time ...
  Resetting $LogFile ... (this might take a while)
  Updating $BadClust file ...
  Updating $Bitmap file ...
  Updating Boot record ...
  Syncing device ...
  Successfully resized NTFS on device '/dev/nvme0n1p3'.
  You can go on to shrink the device for example with Linux fdisk.
  IMPORTANT: When recreating the partition, make sure that you
    1)  create it at the same disk sector (use sector as the unit!)
    2)  create it with the same partition type (usually 7, HPFS/NTFS)
    3)  do not make it smaller than the new NTFS filesystem size
    4)  set the bootable flag for the partition if it existed before
  Otherwise you won't be able to access NTFS or can't boot from the disk!
  If you make a mistake and don't have a partition table backup then you
  can recover the partition table by TestDisk or Parted's rescue mode.
  user@debian ~ % sudo parted /dev/nvme0n1
  GNU Parted 3.5
  Using /dev/nvme0n1
  Welcome to GNU Parted! Type 'help' to view a list of commands.
  (parted) p
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

  (parted) resizepart 3 100000MB
  Warning: Shrinking a partition can cause data loss, are you sure you want to continue?
  Yes/No? yes
  (parted) p
  Model: SAMSUNG MZVL21T0HDLU-00BLL (nvme)
  Disk /dev/nvme0n1: 1024GB
  Sector size (logical/physical): 512B/512B
  Partition Table: gpt
  Disk Flags:

  Number  Start   End     Size    File system  Name                          Flags
   1      1049kB  274MB   273MB   fat32        EFI system partition          boot, hidden, esp
   2      274MB   290MB   16.8MB               Microsoft reserved partition  msftres
   3      290MB   100GB   99.7GB  ntfs         Basic data partition          msftdata
   4      1022GB  1024GB  2097MB  ntfs         Basic data partition          hidden, diag
```

> ℹ️ **Note**: I made a mistake at this point. `parted resize` does not take the size but the **end position** of the partition.
  The size of part 3 was 99.7GB, not 100GB as needed. Windows did not boot and I had to recover the NTFS partition from my backup.

I then created the Debian partitions (boot and LUKS for LVM):

```sh
  (parted) mkpart primary ext2 100GB 101GB
  (parted) mkpart primary 101GB 1022GB
  (parted) p
  Model: SAMSUNG MZVL21T0HDLU-00BLL (nvme)
  Disk /dev/nvme0n1: 1024GB
  Sector size (logical/physical): 512B/512B
  Partition Table: gpt
  Disk Flags:

  Number  Start   End     Size    File system  Name                          Flags
   1      1049kB  274MB   273MB   fat32        EFI system partition          boot, hidden, esp
   2      274MB   290MB   16.8MB               Microsoft reserved partition  msftres
   3      290MB   100GB   99.7GB  ntfs         Basic data partition          msftdata
   5      100GB   101GB   999MB   ext2         primary
   6      101GB   1022GB  921GB                primary
   4      1022GB  1024GB  2097MB  ntfs         Basic data partition          hidden, diag
```

> ℹ️ **Note**: I set the file system to ext2 for the boot partition. I should have choosen ext4, but I don't think this makes any difference.

Then I initialized the LUKS partition:

```sh
  user@debian ~ % sudo cryptsetup luksFormat /dev/nvme0n1p6

  WARNING!
  ========
  This will overwrite data on /dev/nvme0n1p6 irrevocably.

  Are you sure? (Type 'yes' in capital letters): YES
  Enter passphrase for /dev/nvme0n1p6:
  Verify passphrase:

  user@debian ~ % sudo cryptsetup open /dev/nvme0n1p6 nvme0n1p6_crypt
  Enter passphrase for /dev/nvme0n1p6:

  user@debian ~ % sudo pvcreate /dev/mapper/nvme0n1p6_crypt
    Physical volume "/dev/mapper/nvme0n1p6_crypt" successfully created.
```

And set up LVM:

```sh
  user@debian ~ % sudo vgcreate vg0-predator /dev/mapper/nvme0n1p6_crypt
    Volume group "vg0-predator" successfully created

  user@debian ~ % sudo lvcreate -L 16GB -n rootfs vg0-predator
    Logical volume "rootfs" created.
  user@debian ~ % sudo lvcreate -L 20GB -n home vg0-predator
    Logical volume "home" created.

And finally created the file systems:

```sh
  user@debian ~ % sudo mkfs.ext2 /dev/nvme0n1p5
  mke2fs 1.47.0 (5-Feb-2023)
  Discarding device blocks: done
  Creating filesystem with 243968 4k blocks and 61056 inodes
  Filesystem UUID: 3715cd5c-439f-4a43-8ba3-f5ec00d8c4f0
  Superblock backups stored on blocks:
          32768, 98304, 163840, 229376

  Allocating group tables: done
  Writing inode tables: done
  Writing superblocks and filesystem accounting information: done

  user@debian ~ % sudo mkfs.ext4 /dev/mapper/vg0--predator-rootfs
  mke2fs 1.47.0 (5-Feb-2023)
  Creating filesystem with 4194304 4k blocks and 1048576 inodes
  Filesystem UUID: 350aa33c-956c-46ee-a626-be857ac9001c
  Superblock backups stored on blocks:
          32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
          4096000

  Allocating group tables: done
  Writing inode tables: done
  Creating journal (32768 blocks): done
  Writing superblocks and filesystem accounting information: done

  user@debian ~ % sudo mkfs.ext4 /dev/mapper/vg0--predator-home
  mke2fs 1.47.0 (5-Feb-2023)
  Creating filesystem with 5242880 4k blocks and 1310720 inodes
  Filesystem UUID: 61e44127-f0b4-45f6-a043-84cf056f74a5
  Superblock backups stored on blocks:
          32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
          4096000

  Allocating group tables: done
  Writing inode tables: done
  Creating journal (32768 blocks): done
  Writing superblocks and filesystem accounting information: done

  user@debian ~ % sudo e2label /dev/nvme0n1p5 boot
  user@debian ~ % sudo e2label /dev/vg0-predator/rootfs rootfs
  user@debian ~ % sudo e2label /dev/vg0-predator/home home
```

When running the Calamares installer again, I could change the Storage device (at the top) to my PV/VG `vg0-predator`.

From there I could set up the mount points.

![](Screenshot_20230831_203654.png "Partitions - LV home + rootfs")

I also set the `/boot` + `/boot/efi` mount point on the main storage device (`/dev/nvme0n1`):

![](Screenshot_20230831_203740.png "Partitions with Mount Points")

To my suprise I got a warning, that the EFI system partition is configured incorrectly:

![](Screenshot_20230831_203913.png "EFI system partition configured incorrectly")

I started the installation anyway, but it failed:

![](Screenshot_20230831_204407.png "Summary")

![](Screenshot_20230831_205049.png "Installation Failed I")

![](Screenshot_20230831_205053.png "Installation Failed II")

## Debugging the failed installation

I wanted to know why the installation failed. The log looked like this:

```
  Command <i>/usr/sbin/bootloader-config</i> finished with exit code 1.
  Output:
  Running bootloader-config...
   * Installing grub-efi (uefi)...
  Reading package lists...
  Building dependency tree...
  Reading state information...
  The following additional packages will be installed:
    efibootmgr grub-efi-amd64-bin grub-efi-amd64-signed grub2-common mokutil
    shim-helpers-amd64-signed shim-signed shim-signed-common shim-unsigned
  Recommended packages:
    secureboot-db
  The following NEW packages will be installed:
    efibootmgr grub-efi-amd64 grub-efi-amd64-bin grub-efi-amd64-signed
    grub2-common mokutil shim-helpers-amd64-signed shim-signed
    shim-signed-common shim-unsigned
  0 upgraded, 10 newly installed, 0 to remove and 0 not upgraded.
  Need to get 4621 kB of archives.
  After this operation, 40.1 MB of additional disk space will be used.
  Get:1 http://deb.debian.org/debian bookworm/main amd64 efibootmgr amd64 17-2 [27.6 kB]
  Get:2 http://deb.debian.org/debian bookworm/main amd64 grub2-common amd64 2.06-13 [613 kB]
  Get:3 http://deb.debian.org/debian bookworm/main amd64 grub-efi-amd64-bin amd64 2.06-13 [1574 kB]
  Get:4 http://deb.debian.org/debian bookworm/main amd64 grub-efi-amd64 amd64 2.06-13 [45.7 kB]
  Get:5 http://deb.debian.org/debian bookworm/main amd64 grub-efi-amd64-signed amd64 1+2.06+13 [1258 kB]
  Get:6 http://deb.debian.org/debian bookworm/main amd64 mokutil amd64 0.6.0-2 [26.9 kB]
  Get:7 http://deb.debian.org/debian bookworm/main amd64 shim-unsigned amd64 15.7-1 [436 kB]
  Get:8 http://deb.debian.org/debian bookworm/main amd64 shim-helpers-amd64-signed amd64 1+15.7+1 [302 kB]
  Get:9 http://deb.debian.org/debian bookworm/main amd64 shim-signed-common all 1.39+15.7-1 [12.8 kB]
  Get:10 http://deb.debian.org/debian bookworm/main amd64 shim-signed amd64 1.39+15.7-1 [325 kB]
  Preconfiguring packages ...
  Fetched 4621 kB in 1s (5421 kB/s)
  E: Can not write log (Is /dev/pts mounted?) - posix_openpt (19: No such device)
  Selecting previously unselected package efibootmgr.
  (Reading database ... 296064 files and directories currently installed.)
  Preparing to unpack .../0-efibootmgr_17-2_amd64.deb ...
  Unpacking efibootmgr (17-2) ...
  Selecting previously unselected package grub2-common.
  Preparing to unpack .../1-grub2-common_2.06-13_amd64.deb ...
  Unpacking grub2-common (2.06-13) ...
  Selecting previously unselected package grub-efi-amd64-bin.
  Preparing to unpack .../2-grub-efi-amd64-bin_2.06-13_amd64.deb ...
  Unpacking grub-efi-amd64-bin (2.06-13) ...
  Selecting previously unselected package grub-efi-amd64.
  Preparing to unpack .../3-grub-efi-amd64_2.06-13_amd64.deb ...
  Unpacking grub-efi-amd64 (2.06-13) ...
  Selecting previously unselected package grub-efi-amd64-signed.
  Preparing to unpack .../4-grub-efi-amd64-signed_1+2.06+13_amd64.deb ...
  Unpacking grub-efi-amd64-signed (1+2.06+13) ...
  Selecting previously unselected package mokutil.
  Preparing to unpack .../5-mokutil_0.6.0-2_amd64.deb ...
  Unpacking mokutil (0.6.0-2) ...
  Selecting previously unselected package shim-unsigned.
  Preparing to unpack .../6-shim-unsigned_15.7-1_amd64.deb ...
  Unpacking shim-unsigned (15.7-1) ...
  Selecting previously unselected package shim-helpers-amd64-signed.
  Preparing to unpack .../7-shim-helpers-amd64-signed_1+15.7+1_amd64.deb ...
  Unpacking shim-helpers-amd64-signed (1+15.7+1) ...
  Selecting previously unselected package shim-signed-common.
  Preparing to unpack .../8-shim-signed-common_1.39+15.7-1_all.deb ...
  Unpacking shim-signed-common (1.39+15.7-1) ...
  Selecting previously unselected package shim-signed:amd64.
  Preparing to unpack .../9-shim-signed_1.39+15.7-1_amd64.deb ...
  Unpacking shim-signed:amd64 (1.39+15.7-1) ...
  Setting up efibootmgr (17-2) ...
  Setting up mokutil (0.6.0-2) ...
  Setting up grub-efi-amd64-signed (1+2.06+13) ...
  Setting up grub2-common (2.06-13) ...
  Setting up shim-signed-common (1.39+15.7-1) ...
  No DKMS packages installed: not changing Secure Boot validation state.
  Setting up grub-efi-amd64-bin (2.06-13) ...
  Setting up shim-unsigned (15.7-1) ...
  Setting up grub-efi-amd64 (2.06-13) ...

  Creating config file /etc/default/grub with new version
  Installing for x86_64-efi platform.
  Installation finished. No error reported.
  Setting up shim-helpers-amd64-signed (1+15.7+1) ...
  Installing for x86_64-efi platform.
  Installation finished. No error reported.
  Setting up shim-signed:amd64 (1.39+15.7-1) ...
  Installing for x86_64-efi platform.
  Installation finished. No error reported.
  No DKMS packages installed: not changing Secure Boot validation state.
  Processing triggers for man-db (2.11.2-2) ...
  /usr/sbin/grub-probe: error: cannot find a device for / (is /dev mounted?).
```

So, this two lines were interesting:

```
  [...]
  E: Can not write log (Is /dev/pts mounted?) - posix_openpt (19: No such device)
  [...]
  /usr/sbin/grub-probe: error: cannot find a device for / (is /dev mounted?).
```

It turned out that the rootfs (`/dev/vg0-predator/rootfs`) was not mounted into `/tmp/calamares-root-9jen3k4l`.

The following happened to me:

```
  user@debian ~ % l /tmp/calamares-root-9jen3k4l
  total 4
  lrwxrwxrwx   1 root root    7 Jul 22 11:48 bin -> usr/bin
  drwxr-xr-x   5 root root 4096 Aug 31 20:49 boot
  drwxr-xr-x  20 root root 3960 Aug 31 20:45 dev
  drwxr-xr-x 130 root root 4660 Aug 31 20:49 etc
  drwxr-xr-x   3 root root   60 Aug 31 20:49 home
  lrwxrwxrwx   1 root root   30 Jul 22 11:48 initrd.img -> boot/initrd.img-6.1.0-10-amd64
  lrwxrwxrwx   1 root root    7 Jul 22 11:48 lib -> usr/lib
  lrwxrwxrwx   1 root root    9 Jul 22 11:48 lib32 -> usr/lib32
  lrwxrwxrwx   1 root root    9 Jul 22 11:48 lib64 -> usr/lib64
  lrwxrwxrwx   1 root root   10 Jul 22 11:48 libx32 -> usr/libx32
  drwxr-xr-x   2 root root   40 Jul 22 11:48 media
  drwxr-xr-x   2 root root   40 Jul 22 11:48 mnt
  drwxr-xr-x   2 root root   40 Jul 22 11:48 opt
  dr-xr-xr-x 378 root root    0 Aug 31 20:45 proc
  drwx------   4 root root  120 Jul 22 11:48 root
  drwxrwxrwt   4 root root   80 Aug 31 20:48 run
  lrwxrwxrwx   1 root root    8 Jul 22 11:48 sbin -> usr/sbin
  drwxr-xr-x   2 root root   40 Jul 22 11:48 srv
  dr-xr-xr-x  13 root root    0 Aug 31  2023 sys
  drwxrwxrwt   2 root root   40 Aug 31 20:49 tmp
  drwxr-xr-x  14 root root  280 Jul 22 11:48 usr
  drwxr-xr-x  11 root root  260 Jul 22 11:48 var
  lrwxrwxrwx   1 root root   27 Jul 22 11:48 vmlinuz -> boot/vmlinuz-6.1.0-10-amd64

  user@debian ~ % sudo mount --bind /dev /tmp/calamares-root-9jen3k4l/dev
  user@debian ~ % sudo chroot /tmp/calamares-root-9jen3k4l
  sudo: unable to allocate pty: No such device
```

I did not notice that notice that something went wrong with the `/dev` mount point.
I was not able to spawn more another shell to recover from that problem.

I had to reboot.

After that I tried to reproduce the problem.
The system looked like this after the installation:

```sh
  user@debian:~$ lsblk
  NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
  loop0         7:0    0   2.6G  1 loop /tmp/tmp0bzwxuko/filesystem
                                        /usr/lib/live/mount/rootfs/filesystem.squashfs
                                        /run/live/rootfs/filesystem.squashfs
  sda           8:0    1 119.5G  0 disk
  ├─sda1        8:1    1 119.5G  0 part
  └─sda2        8:2    1    32M  0 part
  nvme0n1     259:0    0 953.9G  0 disk
  ├─nvme0n1p1 259:1    0   260M  0 part /tmp/calamares-root-h34xq0uj/boot/efi
  ├─nvme0n1p2 259:2    0    16M  0 part
  ├─nvme0n1p3 259:3    0  92.9G  0 part
  ├─nvme0n1p4 259:4    0     2G  0 part
  ├─nvme0n1p5 259:5    0   953M  0 part /tmp/calamares-root-h34xq0uj/boot
  └─nvme0n1p6 259:6    0 857.9G  0 part
```

So, rootfs and home were not mounted.

I unmounted boot and efi, unlocked the LUKS partition, mounted the rootfs and home LVs manually:

```sh
  user@debian:~$ ls -l /tmp/calamares-root-h34xq0uj/boot
  total 93204
  -rw-r--r-- 1 root root   259507 Jul 14 05:46 config-6.1.0-10-amd64
  drwxr-xr-x 6 root root     4096 Jan  1  1970 efi
  drwxr-xr-x 5 root root     4096 Aug 31 20:49 grub
  -rw-r--r-- 1 root root 87060407 Jul 22 11:48 initrd.img-6.1.0-10-amd64
  drwx------ 2 root root    16384 Aug 31 20:31 lost+found
  -rw-r--r-- 1 root root       83 Jul 14 05:46 System.map-6.1.0-10-amd64
  -rw-r--r-- 1 root root  7981152 Jul 14 05:46 vmlinuz-6.1.0-10-amd64

  user@debian:~$ ls -l /tmp/calamares-root-h34xq0uj/boot/efi/
  total 16
  drwxr-xr-x 2 root root 4096 Jul 24 14:24 '$RECYCLE.BIN'
  drwxr-xr-x 2 root root 4096 Jul 24 14:05  BOOT
  drwxr-xr-x 5 root root 4096 Aug 31 17:30  EFI
  drwxr-xr-x 2 root root 4096 Jul 24 14:16 'System Volume Information'

  user@debian:~$ sudo umount /tmp/calamares-root-h34xq0uj/boot/efi/
  user@debian:~$ sudo umount /tmp/calamares-root-h34xq0uj/boot/
  user@debian:~$ sudo cryptsetup open /dev/nvme0n1p6 nvme0n1p6_crypt
  Enter passphrase for /dev/nvme0n1p6:

  user@debian:~$ sudo pvscan
    PV /dev/mapper/nvme0n1p6_crypt   VG vg0-predator    lvm2 [<857.84 GiB / <821.84 GiB free]
    Total: 1 [<857.84 GiB] / in use: 1 [<857.84 GiB] / in no VG: 0 [0   ]
  user@debian:~$ sudo vgchange -ay vg0-predator
    2 logical volume(s) in volume group "vg0-predator" now active
  [...]
```

But the installation failed again. *sigh*

So I then decided to give up the Calamares installer and use the Debian (netinst)[https://www.debian.org/distrib/netinst] installer.
