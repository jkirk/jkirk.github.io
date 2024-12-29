---
title: "Lenovo ThinkPad X1 Carbon Gen 11: Debian/bookworm Post-Installation Journal"
create: 2023-10-07T08:37:03Z
date: 2023-10-07T08:37:03Z
draft: true
---

Installing Debian is easy. Configuring the system to my needs is much harder.

The goal is to bootstrap/deploy the most important programs and settings as quickly and as automatically as possible.
<!--more-->

## Linux Desktop Bootstrap

To set up my "Linux Desktop" I need my dotfiles and some basic programs.

I developed [linux-desktop-bootstrap.sh](https://github.com/jkirk/linux-desktop-bootstrap), which installs `git`, `etckeeper`, and `ansible-core`.
My dotfiles get deployed and my base software selection gets installed:

```sh
❯ busybox wget -O - https://raw.githubusercontent.com/jkirk/linux-desktop-bootstrap/main/linux-desktop-bootstrap.sh | sh
```

See [jkirk/linux-desktop-bootstrap: Make your GNU/Linux Debian Desktop usable](https://github.com/jkirk/linux-desktop-bootstrap) for details.

## Migrate Data

The next step was to migrate the data. I have several mount points that I copy to an external drive:

```sh
❯ mount -t ext4
/dev/mapper/vg0-rootfs on / type ext4 (rw,relatime,errors=remount-ro)
/dev/mapper/vg0-pbuilder on /var/cache/pbuilder type ext4 (rw,relatime)
/dev/mapper/vg0-isos on /srv/data/isos type ext4 (rw,relatime)
/dev/mapper/vg0-docker on /var/lib/docker type ext4 (rw,relatime)
/dev/mapper/vg0-home on /home type ext4 (rw,relatime)
/dev/mapper/vg0-software on /home/jkirk/software type ext4 (rw,relatime)
/dev/mapper/vg0-vbox on /home/jkirk/VirtualBox VMs type ext4 (rw,relatime)
/dev/mapper/vg0-home.vagrant on /home/jkirk/.vagrant.d type ext4 (rw,relatime)
/dev/mapper/vg0-documents.archive on /home/jkirk/Documents/Archive type ext4 (rw,relatime)
/dev/mapper/vg0-pictures on /home/jkirk/Pictures type ext4 (rw,relatime)
/dev/mapper/vg0-steam on /home/jkirk/software/steam type ext4 (rw,relatime)
/dev/mapper/vg0-android.userdata on /home/jkirk/.android type ext4 (rw,relatime)
/dev/mapper/vg0-projects on /home/jkirk/projects type ext4 (rw,relatime)
/dev/mapper/luks-ba682a60-c9c8-44ec-9c49-420650b96ee3 on /media/jkirk/WORK01 type ext4 (rw,nosuid,nodev,relatime,errors=remount-ro,uhelper=udisks2)
```

```sh
❯ sudo rsync -avxHAX --progress --delete /home/jkirk /home/jkirk/projects /home/jkirk/software /home/jkirk/Documents/Archive /home/jkirk/Pictures /home/jkirk/software /etc /media/jkirk/WORK01/backup.executor.20231005
```

## Settings

### Change hostname + LVM Volume Group Name

After the installation, I decided to change the hostname from `predator` to `tranquility`:

```sh
❯ sudo hostnamectl hostname tranquilit

❯ sudo etckeeper vcs show 04c5a263fc10ae81061e5426b3d434a1d328363a /etc/hosts /etc/hostname
commit 04c5a263fc10ae81061e5426b3d434a1d328363a
Author: Darshaka Pathirana <dpat@syn-net.org>
Date:   Sat Oct 7 12:10:43 2023 +0200

    Change hostname from predator to tranquility

diff --git a/hostname b/hostname
index 3dc1eee..97e91f3 100644
--- a/hostname
+++ b/hostname
@@ -1 +1 @@
-predator
+tranquility
diff --git a/hosts b/hosts
index 684d4e6..a5431c8 100644
--- a/hosts
+++ b/hosts
@@ -1,5 +1,5 @@
 127.0.0.1      localhost
-127.0.1.1      predator.home.syn-net.org       predator
+127.0.1.1      tranquility.home.syn-net.org    tranquility

 # The following lines are desirable for IPv6 capable hosts
 ::1     localhost ip6-localhost ip6-loopback
```

The LVM Volume Group Name `vg0-predator` also needs to be renamed to `vg0-tranquility`.

This was a bit trickier as the X server / Display Manager was restarted when I did the following:

```sh
❯ sudo vgrename vg0-predator vg0-tranquility
```

I adjusted `/etc/fstab` + `/boot/grub/grub.cfg` and updated initramfs:

```sh
❯ sudo etckeeper vcs show 04c5a263fc10ae81061e5426b3d434a1d328363a /etc/fstab
commit 04c5a263fc10ae81061e5426b3d434a1d328363a
Author: Darshaka Pathirana <dpat@syn-net.org>
Date:   Sat Oct 7 12:10:43 2023 +0200

    Change hostname from predator to tranquility

diff --git a/fstab b/fstab
index a1b23ff..18edcc6 100644
--- a/fstab
+++ b/fstab
@@ -8,9 +8,9 @@
 # Please run 'systemctl daemon-reload' after making changes here.
 #
 # <file system> <mount point>   <type>  <options>       <dump>  <pass>
-/dev/mapper/vg0--predator-rootfs /               ext4    errors=remount-ro 0       1
+/dev/mapper/vg0--tranquility-rootfs /               ext4    errors=remount-ro 0       1
 # /boot was on /dev/nvme0n1p5 during installation
 UUID=4b42a4ed-14f5-42c4-b006-9f1ceca7ad11 /boot           ext4    defaults        0       2
 # /boot/efi was on /dev/nvme0n1p1 during installation
 UUID=50D7-B9E5  /boot/efi       vfat    umask=0077      0       1
-/dev/mapper/vg0--predator-home /home           ext4    defaults        0       2
+/dev/mapper/vg0--tranquility-home /home           ext4    defaults        0       2

❯ sudo grep rootfs /boot/grub/grub.cfg
        linux   /vmlinuz-6.1.0-12-amd64 root=/dev/mapper/vg0--tranquility-rootfs ro  quiet
                linux   /vmlinuz-6.1.0-12-amd64 root=/dev/mapper/vg0--tranquility-rootfs ro  quiet
                linux   /vmlinuz-6.1.0-12-amd64 root=/dev/mapper/vg0--tranquility-rootfs ro single
                linux   /vmlinuz-6.1.0-10-amd64 root=/dev/mapper/vg0--tranquility-rootfs ro  quiet
                linux   /vmlinuz-6.1.0-10-amd64 root=/dev/mapper/vg0--tranquility-rootfs ro single

❯ sudo update-initramfs -u -k all
```

## Networking / NetworkManager

### NetworkManager Profiles

To migrate the network settings, I had to update the interface name and remove the 'mac-address' lines:

```
❯ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: wwan0: <POINTOPOINT,NOARP> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/none
3: wlp0s20f3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff permaddr f4:3b:d8:a9:f0:97
```

```sh
❯ sudo cp -a /mnt/etc/NetworkManager/system-connections/. /etc/NetworkManager/system-connections/.
❯ sudo sed -i -e "s/interface-name=wlp59s0/interface-name=wlp0s20f3/" /etc/NetworkManager/system-connections/*
❯ sudo sed -e '/mac-address=.*/d' -i /etc/NetworkManager/system-connections/*
```

Note, that everything except `mode` and `ssid` in the `[wifi]` section is not needed.
The following works fine for me:

```
[wifi]
mode=infrastructure
ssid=home.syn-net.org
```

### NetworkManager DNS + VPN

Wireguard actually works out of the box, a network manager plugin is not needed to import the wireguard profile:

```
❯ nmcli connection import type wireguard file wireguard/wg-hetzner.conf
Connection 'wg-hetzner' (a7755f75-af8e-4cd9-965b-e0ea2410c9af) successfully added.
```

See: https://blogs.gnome.org/thaller/2019/03/15/wireguard-in-networkmanager/

> **_NOTE:_** The private key which is saved in the profile file is needed to connect to Wireguard peers.
>
> To generate a new private / public key pair, one should use `wg genkey` from the `wireguard-tools` package.
>
> To derive the public key from the private key run `wg pubkey < privatekey > publickey`.

I use different DNS servers for different VPNs.
I.e. I use the "internal" DNS server for my "internal" VMs on the Hetzner server.
For that I have a dnsmasq configuration like this (see [below](#networkmanager-with-dnsmasq) for details)

```sh
❯ cat /etc/NetworkManager/dnsmasq.d/hetzner.conf
server=/h2.syn-net.org/10.10.1.1
```

I also tried the DNS setting in NetworkManager WireGuard profile:

```sh
❯ nmcli c modify wg-hetzner ipv4.dns 10.10.1.1

❯ nmcli c show wg-hetzner | grep ipv4.dns:
ipv4.dns:                               10.10.1.1
```

This works as well but leaks the VPN domain name to the external DNS server and or external queries to the VPN DNS server.

I reverted this setting with: `nmcli c modify wg-home-all ipv4.dns ""`.

I also tried to import the OpenVPN setting, but got an error message:

```sh
❯ nmcli connection import type openvpn file tmp/openvpn/myopenvpn.conf
Error: failed to import 'tmp/openvpn/myopenvpn.conf': configuration error: unsupported 1th argument remote_host to “route” (line 6).

❯ grep "route " tmp/openvpn/myopenvpn.conf
route remote_host 255.255.255.255 net_gateway
````

I removed the line in question. Its a bug in network-manager: https://bugs.launchpad.net/ubuntu/+source/network-manager-openvpn/+bug/606365/comments/68 + https://askubuntu.com/a/1013116

Importing the setting worked fine afterwards:

```sh
❯ nmcli connection import type openvpn file tmp/openvpn/myopenvpn.conf
Connection 'myopenvpn' (62875c23-aadf-4d1c-896c-65e14cdb9a5e) successfully added.
````

I connected the VPN (via the NetworkManager GUI) and was prompted for the VPN password and saved it:

```sh
❯ nmcli c show --show-secrets myopenvpn | grep vpn.secret
vpn.secrets:                            password = [SNIP]
```

There still was problem with some connections:

```
Oct 16 09:08:20 tranquility nm-openvpn[86980]: VERIFY ERROR: depth=0, error=CA signature digest algorithm too weak: C=at, [SNIP]
Oct 16 09:08:20 tranquility nm-openvpn[86980]: OpenSSL: error:0A000086:SSL routines::certificate verify failed
Oct 16 09:08:20 tranquility nm-openvpn[86980]: TLS_ERROR: BIO read tls_read_plaintext error
Oct 16 09:08:20 tranquility nm-openvpn[86980]: TLS Error: TLS object -> incoming plaintext read error
Oct 16 09:08:20 tranquility nm-openvpn[86980]: TLS Error: TLS handshake failed
Oct 16 09:08:20 tranquility nm-openvpn[86980]: Fatal TLS error (check_tls_errors_co), restarting
```

```
Nov 09 15:15:52 tranquility NetworkManager[2610319]: 2023-11-09 15:15:52 WARNING: Compression for receiving enabled. Compression has been used in the past to break encryption. Sent packets are not compressed unless "allow-compression yes" is also set.
Nov 09 15:15:52 tranquility nm-openvpn[2610319]: OpenVPN 2.6.3 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] [DCO]
Nov 09 15:15:52 tranquility nm-openvpn[2610319]: library versions: OpenSSL 3.0.11 19 Sep 2023, LZO 2.10
Nov 09 15:15:52 tranquility nm-openvpn[2610319]: DCO version: N/A
Nov 09 15:15:52 tranquility nm-openvpn[2610319]: WARNING: No server certificate verification method has been enabled.  See http://openvpn.net/howto.html#mitm for more info.
Nov 09 15:15:52 tranquility nm-openvpn[2610319]: NOTE: the current --script-security setting may allow this configuration to call user-defined scripts
Nov 09 15:15:52 tranquility nm-openvpn[2610319]: OpenSSL: error:0A00018E:SSL routines::ca md too weak
Nov 09 15:15:52 tranquility nm-openvpn[2610319]: Cannot load certificate file /home/jkirk/.cert/nm-openvpn/snr.user.crt
Nov 09 15:15:52 tranquility nm-openvpn[2610319]: Exiting due to fatal error
```

I had to change the following setting:

In Network Setting > VPN connection myopenvpn > Identiy > Advanced > TLS Authentication > Additional TLS authentication or encryption

* TLS cipher string: `DEFAULT:@SECLEVEL=0`

See:

* https://superuser.com/a/1737054
* https://superuser.com/a/1741782

This setting is only needed for older OpenVPN servers.

I then disabled autoconnect + set "use this connection only for resources on its network":

```sh
❯ nmcli c modify myopenvpn connection.autoconnect no
❯ nmcli c modify myopenvpn ipv4.never-default yes
```

### NetworkManager with dnsmasq

I use NetworkManager together with dnsmasq to be able to query different DNS servers for different domains.

I migrated my dnsmasq settings:

```
❯ sudo cp -a /mnt/etc/NetworkManager/dnsmasq.d/*.conf /etc/NetworkManager/dnsmasq.d
❯ sudo cp -a /mnt/etc/NetworkManager/conf.d/dnsmasq.conf /etc/NetworkManager/conf.d

❯ cat /etc/NetworkManager/conf.d/dnsmasq.conf
[main]
dns=dnsmasq
```

A DNS server for the "internal" domain looks like this:

```
❯ cat /etc/NetworkManager/dnsmasq.d/helios.conf
server=/h2.syn-net.org/10.10.1.1
```

Queries for `h2.syn-net.org` go to the DNS server 10.10.1.1.

No DNS should be set in the VPN settings.

### NetworkManager Auto Connection:

If the autoconnect is set to true, the NetworkManager activates the connection automatically.

To check auto-connection:

```sh
❯ nmcli c show wg-hetzner | grep connection.autoconnect:
connection.autoconnect:                 yes
```

I noticed that the VPN DNS take priority over the globally set on.
But if multiple VPN connections are enabled, the order from which they are started makes a difference.
I will have to investigate the "ipv4.dns-priority" setting.

However, as I use dnsmasq for my VPN domains, I usually have do not have a DNS server set up for my VPN connections.

### NetworkManager Mobile Broadband

I tried to setup Mobile Broadband but the NetworkManager bug seems still to exist:

* linuxmint/cinnamon-control-center#225: Network Manager crashes when connecting to cellular network· https://github.com/linuxmint/cinnamon-control-center/issues/225

When trying to set up a the mobile broadband manually, I noticed that there is no primary port name starting with "tty" and state is "disabled":

```sh
❯ mmcli -L
    /org/freedesktop/ModemManager1/Modem/5 [Intel] MBIM [8086:7560]

❯ mmcli -m 5
  ----------------------------------
  General  |                   path: /org/freedesktop/ModemManager1/Modem/5
           |              device id: 725df593632bb5ad0f70fb16ab75746592cf9228
  ----------------------------------
  Hardware |           manufacturer: Intel
           |                  model: MBIM [8086:7560]
           |      firmware revision: 18601.5001.00.01.01.32_GC
           |           h/w revision: V1.3
           |              supported: gsm-umts, lte
           |                current: gsm-umts, lte
           |           equipment id: 016175003144831
  ----------------------------------
  System   |                 device: /sys/devices/pci0000:00/0000:00:1c.0/0000:08:00.0
           |                drivers: iosm
           |                 plugin: Intel
           |           primary port: wwan0mbim0
           |                  ports: wwan0 (net), wwan0at0 (at), wwan0at1 (at), wwan0mbim0 (mbim)
  ----------------------------------
  Status   |                  state: disabled
           |            power state: low
  ----------------------------------
  Modes    |              supported: allowed: 3g, 4g; preferred: none
           |                current: allowed: 3g, 4g; preferred: none
  ----------------------------------
  IP       |              supported: ipv4, ipv6, ipv4v6
  ----------------------------------
  3GPP     |                   imei: 016175003144831
           |   packet service state: detached
  ----------------------------------
  3GPP EPS |   ue mode of operation: csps-2
           | initial bearer ip type: ipv4
  ----------------------------------
  SIM      |       primary sim path: /org/freedesktop/ModemManager1/SIM/5
```

Tried to enable modem:

```sh
❯ mmcli -m 5 --enable
error: couldn't enable the modem: 'GDBus.Error:org.freedesktop.ModemManager1.Error.Core.Retry: Invalid transition'
```

Did some research:

```sh
❯ lspci -s 08:00.0 -vv
08:00.0 Wireless controller [0d40]: Intel Corporation XMM7560 LTE Advanced Pro Modem (rev 01)
        Subsystem: Device 1cf8:8654
        Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx+
        Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
        Latency: 0
        Interrupt: pin A routed to IRQ 142
        IOMMU group: 17
        Region 0: Memory at bc200000 (64-bit, non-prefetchable) [size=4K]
        Region 2: Memory at bc201000 (64-bit, non-prefetchable) [size=256]
        Capabilities: <access denied>
        Kernel driver in use: iosm
        Kernel modules: iosm

❯ lspci -s 08:00.0 -nnk
08:00.0 Wireless controller [0d40]: Intel Corporation XMM7560 LTE Advanced Pro Modem [8086:7560] (rev 01)
        Subsystem: Device [1cf8:8654]
        Kernel driver in use: iosm
        Kernel modules: iosm
```

The hardware itself seems to be supported:

* Intel XMM7560 LTE Advanced Pro Modem: https://linux-hardware.org/?id=pci:8086-7560-103c-893b
* L860-GL-16-Leading 5G Wireless Modules & IoT Solutions | Fibocom: https://www.fibocom.com/en/products/LTECat-L860-GL-16.html

Fibocom L860-GL and Intel XMM7560 seem to be the same, but I am not completely sure:

* HOWTO - Mobile Broadband - Fibocom L860-GL - xmm 7560 / Laptop Issues / Arch Linux Forums: https://bbs.archlinux.org/viewtopic.php?id=282722

The Fibocom L860 needs a FCC unlock procedure?!:

* Lenovo ThinkPad X1 Carbon (Gen 10) - ArchWiki: https://wiki.archlinux.org/title/Lenovo_ThinkPad_X1_Carbon_(Gen_10)
* https://wiki.archlinux.org/title/Lenovo_ThinkPad_X1_Carbon_(Gen_9)#Mobile_broadband
* FCC unlock procedure | ModemManager: https://modemmanager.org/docs/modemmanager/fcc-unlock/#fcc-unlock-procedures-in-modemmanager--1184-1
* AUR (en) - thinkpad-l860-gl-fcc-unlock-bin: https://aur.archlinux.org/packages/thinkpad-l860-gl-fcc-unlock-bin
* Fibocom Wireless WAN L860-GL-16 FCC Unlock and SAR Config tool for Linux - ThinkPad: https://pcsupport.lenovo.com/au/en/products/laptops-and-netbooks/thinkpad-x-series-laptops/thinkpad-x1-carbon-11th-gen-type-21hm-21hn/downloads/ds563599-fibocom-wireless-wan-l860-gl-16-fcc-unlock-and-sar-config-tool-for-linux-thinkpad?category=Networking%3A+Wireless+WAN
* https://download.lenovo.com/pccbbs/mobiles/n3xwp01w.txt

There is a promising ModemManager issue:

* https://gitlab.freedesktop.org/mobile-broadband/ModemManager/-/issues/770
* https://bbs.archlinux.org/viewtopic.php?pid=2118957#p2118957

* Fibocom L860 Force Hangup (#258) · Issues · Mobile broadband connectivity / ModemManager · GitLab: https://gitlab.freedesktop.org/mobile-broadband/ModemManager/-/issues/258#note_672339

  * https://gitlab.freedesktop.org/mobile-broadband/ModemManager/-/issues/258#note_1108701
  * https://gitlab.freedesktop.org/mobile-broadband/ModemManager/-/issues/385

Other links:

* Related: Driver for Fibocom L850-GL / Intel XMM7360 (PCI ID 8086:7360): https://github.com/xmm7360/xmm7360-pci/pull/50

Unfortunately I could not get the modem to work.
I will investigate the problem later.

### NetworkManager: read / share WIFI via QR code

I found this nifty tool, that allows me to share the current + and share the WIFI connection via QR code:

*  kokoye2007/wifi-qr: Wifi QR code create and scan for linux: https://github.com/kokoye2007/wifi-qr

```
wifi-qr s
```

## Cinnamon Settings

I noticed that some of the settings were not loaded immediately/automatically (although they should be).
I noticed this by any chance with the following shortcuts.

What helped, was to delete the setting / unregister the keyboard shortcut and load the (specific) setting again.

One can monitor the changes to the dconf database with `dconf watch PATH`

The underlying problem is, that the keyboard shortcuts are not listed in the `custom-list`:

```
    ❯ dconf list /org/cinnamon/desktop/keybindings/custom-keybindings/
    custom0/
    custom1/
    custom2/
    custom3/
    custom4/
    custom5/
    custom6/
    custom7/
    custom8/

    ❯ dconf read /org/cinnamon/desktop/keybindings/custom-list
    ['custom8', 'custom0', 'custom1', 'custom2', 'custom3', 'custom5', 'custom6', 'custom7']
```

I could not figure out, why keybinding is lost from the custom-list.

### Cinnamon Keyboard Shortcuts

TODO: transfer file:

```
❯ dconf read /org/cinnamon/desktop/keybindings/custom-list
['custom4', 'custom7', 'custom6', 'custom5', 'custom3', 'custom2', 'custom1', 'custom0', 'custom8']
```

To backup Cinnamon Setting one has to use `dconf dump`.

Over the years some unintentional / unneeded settings have accumulated.
So I did not want to restore every setting I have set in Cinnamon.

To list the old settings, I copied the old dconf database next to the new one.

```sh
❯ cp /mnt/jkirk/.config/dconf/user .config/dconf/user_executor
```

In `.config/dconf/profile` I created a new "dconf profile" `executor` where I set the user `dconf database` file `user_executor`:

```sh
❯ mkdir .config/dconf/profile
❯ cat .config/dconf/profile/executor
user-db:user_executor
```

With the `DCONF_PROFILE` environment variable set, dconf will attempt to open the named profile.

For example, to list the Window Manager keybindings of my current (empty) my old system, I can list them like this:

```sh
❯ dconf dump /org/cinnamon/desktop/keybindings/wm/

❯ DCONF_PROFILE=/home/jkirk/.config/dconf/profile/executor dconf dump /org/cinnamon/desktop/keybindings/wm/
[/]
lower=['<Shift><Alt>m']
maximize=@as []
maximize-vertically=['<Super>v']
minimize=['<Super>m']
move-to-center=['<Primary><Super>j']
move-to-corner-ne=['<Shift><Super>j']
move-to-corner-nw=@as []
move-to-corner-se=@as []
move-to-corner-sw=@as []
move-to-monitor-down=['<Super><Shift>Down', '<Primary><Alt>s']
move-to-monitor-left=['<Super><Shift>Left', '<Primary><Alt>z']
move-to-monitor-right=['<Primary><Alt>x', '<Shift><Super>Right']
move-to-monitor-up=['<Super><Shift>Up', '<Primary><Alt>a']
move-to-workspace-1=['<Primary><Shift><Alt>exclam']
move-to-workspace-2=['<Primary><Shift><Alt>at']
move-to-workspace-3=['<Primary><Shift><Alt>numbersign']
move-to-workspace-4=['<Primary><Shift><Alt>asciitilde', '<Primary><Shift><Alt>dollar']
move-to-workspace-left=['<Control><Shift><Alt>Left']
move-to-workspace-right=['<Control><Shift><Alt>Right']
push-tile-down=['<Super>Down', '<Super>j']
push-tile-left=['<Super>Left', '<Super>h']
push-tile-right=['<Super>Right', '<Super>l']
push-tile-up=['<Super>Up', '<Super>k']
switch-to-workspace-1=['<Super>F1', '<Primary><Alt>1']
switch-to-workspace-2=['<Super>F2', '<Primary><Alt>2']
switch-to-workspace-3=['<Super>F3', '<Primary><Alt>3']
switch-to-workspace-4=['<Super>F4', '<Primary><Alt>4']
switch-to-workspace-down=['<Control><Alt>Down', '<Super>Tab']
switch-to-workspace-left=['<Primary><Alt>Left', '<Primary><Alt>h']
switch-to-workspace-right=['<Control><Alt>Right', '<Primary><Alt>l']
toggle-above=['<Super>a']
toggle-maximized=['<Alt>F10', '<Super>f']
toggle-on-all-workspaces=['<Primary><Alt>grave', '<Primary><Shift><Alt>a']
```

To set a specific value:

```sh
❯ dconf write /org/cinnamon/desktop/keybindings/wm/lower "['<Shift><Alt>m']"
```

To migrate a subset of settings:

```sh
❯ DCONF_PROFILE=/home/jkirk/.config/dconf/profile/executor dconf dump /org/cinnamon/desktop/keybindings/ > tmp/dconf.keybindings
❯ vim tmp/dconf.keybindings
❯ dconf load /org/cinnamon/desktop/keybindings/ < tmp/dconf.keybindings
```

See:

* dconf(1)
* dconf(7)

### Cinnamon Keyboard Settings

```sh
❯ dconf dump /org/gnome/libgnomekbd/keyboard/
[/]
layouts=['us\taltgr-intl', 'at\tnodeadkeys', 'us']
options=['grp\tgrp:shift_caps_toggle']

❯ DCONF_PROFILE=/home/jkirk/.config/dconf/profile/executor dconf dump /org/gnome/libgnomekbd/keyboard/
[/]
layouts=['us\taltgr-intl', 'at\tnodeadkeys', 'us']
options=['grp\tgrp:shift_caps_toggle']
```

### Cinnamon Settings: gnome-screenshot

Transferred the settings:

```sh
❯ DCONF_PROFILE=/home/jkirk/.config/dconf/profile/executor dconf dump /org/gnome/gnome-screenshot/
[/]
auto-save-directory='file:///home/jkirk/Pictures/screenshots'
border-effect='none'
delay=3
include-border=true
include-pointer=false
last-save-directory='file:///home/jkirk/Pictures/screenshots'

❯ DCONF_PROFILE=/home/jkirk/.config/dconf/profile/executor dconf dump /org/gnome/gnome-screenshot/ | dconf load /org/gnome/gnome-screenshot/
```

### Cinnamon Settings: gnome-terminal

Transferred the settings:

```sh
❯ DCONF_PROFILE=/home/jkirk/.config/dconf/profile/executor dconf dump /org/gnome/terminal/legacy/ | dconf load /org/gnome/terminal/legacy/
```

### Cinnamon Settings: Diodon

Transferred the settings:

```sh
❯ DCONF_PROFILE=/home/jkirk/.config/dconf/profile/executor dconf dump /net/launchpad/diodon/ | dconf load /net/launchpad/diodon/
```

The problem with the menu popup seems to fixed: https://bugs.launchpad.net/diodon/+bug/1630375

Changed the delayed start:

```sh
❯ DCONF_PROFILE=/home/jkirk/.config/dconf/profile/executor dconf dump /org/cinnamon/desktop/keybindings/custom-keybindings/custom4/
[/]
binding=['<Shift><Super>c']
command='/home/jkirk/bin/diodon-delay'
name='show clipboard'

❯ dconf dump /org/cinnamon/desktop/keybindings/custom-keybindings/custom4/
[/]
binding=['<Shift><Super>c']
command='/usr/bin/diodon'
name='Diodon'
```

### Cinnamon Settings: Windows / Alt-Tab

Changed the Alt-Tab switcher style from "Icons and thumbnails" to "Icons and window preview"

```
  ❯ dconf read /org/cinnamon/alttab-switcher-style
  'icons+preview'
```

### Cinnamon Settings: Theme

![](screenshot_20231007T160540.png "Default Theme of GNOME Terminal")

I had the following settings in Debian bullseye

![](screenshot_20231007T115946.png "Cinnamon Themes on Debian/bullseye")

The same settings looked like this in Debian/bookworm:

![](screenshot_20231007T160332.png "Cinnamon Themes on Debian/bookworm")

I like a dark theme like Adapta-Nokoto, so after installing the theme I changed the Desktop + Application theme setting to `Adapta-Nokoto`.

![](screenshot_20231007T163736.png "GNOME Terminal in Adapta-Nokoto Theme")

### Cinnamon Settings: Nemo

```
❯ DCONF_PROFILE=/home/jkirk/.config/dconf/profile/executor dconf dump /org/nemo/
[desktop]
computer-icon-visible=false
desktop-layout='true::false'
show-orphaned-desktop-icons=false

[list-view]
search-visible-columns=['name', 'size', 'type', 'where']

[plugins]
disabled-actions=@as []

[preferences]
date-format='iso'
default-folder-viewer='list-view'
ignore-view-metadata=true
show-compact-view-icon-toolbar=true
show-computer-icon-toolbar=false
show-hidden-files=true
show-list-view-icon-toolbar=true
show-location-entry=false
show-open-in-terminal-toolbar=true
start-with-dual-pane=true

[window-state]
bookmarks-expanded=true
devices-expanded=true
geometry='959x500+0+0'
maximized=false
my-computer-expanded=true
network-expanded=true
side-pane-view='places'
sidebar-bookmark-breakpoint=3
sidebar-width=254
start-with-sidebar=true

❯ DCONF_PROFILE=/home/jkirk/.config/dconf/profile/executor dconf dump /org/nemo/ | dconf load /org/nemo/
```

Disable media handling:

```
❯ DCONF_PROFILE=/home/jkirk/.config/dconf/profile/executor dconf dump /org/cinnamon/desktop/media-handling/
[/]
automount=false
automount-open=false
autorun-never=true
autorun-x-content-ignore=['x-content/audio-player', 'x-content/unix-software', 'x-content/image-dcf']
autorun-x-content-open-folder=['x-content/bootable-media']
autorun-x-content-start-app=['x-content/audio-player', 'x-content/image-dcf']

❯ DCONF_PROFILE=/home/jkirk/.config/dconf/profile/executor dconf dump /org/cinnamon/desktop/media-handling/ | grep -v autorun-x | dconf load /org/cinnamon/desktop/media-handling/
```

### Cinnamon Settings: Applets

The default applets:

```
❯ dconf read /org/cinnamon/enabled-applets
['panel1:left:0:menu@cinnamon.org:0', 'panel1:left:1:separator@cinnamon.org:1', 'panel1:left:2:grouped-window-list@cinnamon.org:2', 'panel1:right:0:systray@cinnamon.org:3', 'panel1:right:1:xapp-status@cinnamon.org:4', 'panel1:right:2:notifications@cinnamon.org:5', 'panel1:right:3:printers@cinnamon.org:6', 'panel1:right:4:removable-drives@cinnamon.org:7', 'panel1:right:5:keyboard@cinnamon.org:8', 'panel1:right:6:favorites@cinnamon.org:9', 'panel1:right:7:network@cinnamon.org:10', 'panel1:right:8:sound@cinnamon.org:11', 'panel1:right:9:power@cinnamon.org:12', 'panel1:right:10:calendar@cinnamon.org:13', 'panel1:right:11:cornerbar@cinnamon.org:14']
```

Make it more readable:

```
❯ dconf read /org/cinnamon/enabled-applets | tr -d '\]' | tr -d '\[' | tr ',' '\n' | tr -d ' '
'panel1:left:0:menu@cinnamon.org:0'
'panel1:left:1:separator@cinnamon.org:1'
'panel1:left:2:grouped-window-list@cinnamon.org:2'
'panel1:right:0:systray@cinnamon.org:3'
'panel1:right:1:xapp-status@cinnamon.org:4'
'panel1:right:2:notifications@cinnamon.org:5'
'panel1:right:3:printers@cinnamon.org:6'
'panel1:right:4:removable-drives@cinnamon.org:7'
'panel1:right:5:keyboard@cinnamon.org:8'
'panel1:right:6:favorites@cinnamon.org:9'
'panel1:right:7:network@cinnamon.org:10'
'panel1:right:8:sound@cinnamon.org:11'
'panel1:right:9:power@cinnamon.org:12'
'panel1:right:10:calendar@cinnamon.org:13'
'panel1:right:11:cornerbar@cinnamon.org:14'
```

## Other Software settings

### taskwarrior

Migrated taskwarrior data location:

```
❯ cp -a /mnt/jkirk/Documents/taskwarrior/.task .task
```

And migrated the taskrc configuration file from /mnt/jkirk/Documents/taskwarrior to dotfiles.

See: Merge unrelated git histories.

```
(1/5) 'Writeable' context
  [...]

  What do I have to do?
  You have 9 defined contexts, out of which 9 are old-style:
  * als: proj:sp.als
  * home: proj.not:sp
  * lw: proj:lw
  * notals: proj.not:sp.als
  * oebm: tags:oebm
  * pmt: proj:sp.pmt
  * prt: proj:sp.prt
  * waldrapp: proj:sn.waldrapp
  * work: project.has:sn or project.has:sp

  These need to be migrated to new-style, which uses context.<name>.read and
  context.<name>.write config variables. Please run the following commands:
  $ task context define als 'proj:sp.als'
  $ task context define home 'proj.not:sp'
  $ task context define lw 'proj:lw'
  $ task context define notals 'proj.not:sp.als'
  $ task context define oebm 'tags:oebm'
  $ task context define pmt 'proj:sp.pmt'
  $ task context define prt 'proj:sp.prt'
  $ task context define waldrapp 'proj:sn.waldrapp'
  $ task context define work 'project.has:sn or project.has:sp'

  Please check these filters are also valid modifications. If a context filter is not
  a valid modification, you can set the context.<name>.write configuration variable to
  specify the write context explicitly. Read more in CONTEXT section of man taskrc.

(2/5) Deprecation of the status:waiting

  Background
  If a task has a 'wait' attribute set to a date in the future, it is modified
  to have a 'waiting' status. Once that date is no longer in the future, the status
  is modified to back to 'pending'.

  What changed in 2.6.0?
  The 'waiting' value of status is deprecated, instead users should use +WAITING
  virtual tag, or explicitly query for wait.after:now (the two are equivalent).

  The status:waiting query still works in 2.6.0, but support will be dropped in 3.0.

  What do I have to do?
  In your custom report definitions, the following expressions should be replaced:
  * 'status:pending or status:waiting' should be replaced by 'status:pending'
  * 'status:pending' should be replaced by 'status:pending -WAITING'


(3/5) Environment variables in the taskrc

  What changed in 2.6.0?
  Taskwarrior now supports expanding environment variables in the taskrc file,
  allowing users to customize the behaviour of 'task' based on the current env.

  The environment variables can either be used in paths, or as separate values:
    data.location=$XDG_DATA_HOME/task/
    default.project=$PROJECT

(4/5) Context-less reports

  Background
  By default, every report is affected by currently active context.

  What changed in 2.6.0?
  You can now make a selected report ignore currently active context by setting
  'report.<name>.context' configuration variable to 0.

  What was the motivation behind this feature?
  This is useful for users who utilize a single place (such as project:Inbox)
  to collect their new tasks that are then triaged on a regular basis
  (such as in GTD methodology).

  In such a case, defining a report that filters for project:Inbox and making it
  fully accessible from any context is a major usability improvement.


(5/5) Support for XDG Base Directory Specification

  Background
  The XDG Base Directory specification provides standard locations to store
  application data, configuration, state, and cached data in order to keep $HOME
  clutter-free. The locations are usually set to ~/.local/share, ~/.config,
  ~/.local/state and ~/.cache respectively.

  What changed in 2.6.0?
  If taskrc is not found at '~/.taskrc', Taskwarrior will attempt to find it
  at '$XDG_CONFIG_HOME/task/taskrc' (defaults to '~/.config/task/taskrc').

  What was the motivation behind this feature?
  This allows users to fully follow XDG Base Directory Spec by moving their taskrc:
      $ mkdir $XDG_CONFIG_HOME/task
      $ mv ~/.taskrc $XDG_CONFIG_HOME/task/taskrc

  and further setting:
      data.location=$XDG_DATA_HOME/task/
      hooks.location=$XDG_CONFIG_HOME/task/hooks/

  Solutions in the past required symlinks or more cumbersome configuration overrides.

  What do I have to do?
  If you configure your data.location and hooks.location as above, ensure
  that the XDG_DATA_HOME and XDG_CONFIG_HOME environment variables are set,
  otherwise they're going to expand to empty string. Alternatively you can
  hardcode the desired paths on your system.

```

Run 'task news 2.6.0 minor' for more.

Run 'task news 2.6.0'.


### VIM



```
error detected while processing modelines:
line 4207:
E992: Not allowed in a modeline when 'modelineexpr' is off: foldtext=getline(v:foldstart).'...'.(v:foldend-v:foldstart)
```

Check modelines settings:

```
:set modeline?
:set modelines?
:set modelinexpr?
```

modeline if off by default in Debian, see:


### Firefox Quantum

I used to have Firefox Quantum in `$HOME/software/firefox`.
I decided to migrate it to `/opt`:

```
❯ sudo cp -a /mnt/software/firefox /opt
```

Created a Cinnamon Menu entry:

```sh
❯ cat .local/share/applications/alacarte-made-32f9f494-64ff-11ee-9c1d-f43bd8a9f097.desktop
[Desktop Entry]
Name=Firefox Quantum
Exec=/opt/firefox/firefox --new-instance %u
Comment=
Terminal=false
Icon=/opt/firefox/browser/chrome/icons/default/default128.png
Type=Application
```

Changed the default browser by starting firefox and setting the default browser.

```sh
❯ xdg-settings get default-web-browser
userapp-Firefox-N2TWC2.desktop

❯ cat .local/share/applications/userapp-Firefox-N2TWC2.desktop
[Desktop Entry]
Encoding=UTF-8
Version=1.0
Type=Application
NoDisplay=true
Exec=/opt/firefox/firefox-bin %u
Name=Firefox
Comment=Custom definition for Firefox
```

### Firefoxx + KeepassXC

Transferred the settings file

```sh
❯ cp -a /mnt/jkirk/.config/keepassxc .config
```

One also have to copy `.mozilla/native-messaging-hosts` or "Enable browser integration" for Firefox in KeePassXC > Browser Integration.

### Chromium



```sh
❯ cp -a /mnt/jkirk/.config/chromium .config
❯ cp -a /mnt/jkirk/.cache/chromium .cache
```

### Thunderbird: Profiles + Settings

My Thunderbird Profile is more than 10 years old.
I decided to create a new one from scatch.

I have a *lot* of mail accounts + identiies. How to import/export them?

```
❯ grep -e "mail\.server\.server[0-9]\+\.name" /mnt/jkirk/.thunderbird/bbq2wowx.default/prefs.js | wc -l
26
```

```sh
❯ grep -e "mail\.identity\.id[0-9]\+\.useremail" /mnt/jkirk/.thunderbird/bbq2wowx.default/prefs.js | wc -l
29
```

`mail.server` + `mail.identity` are the lines in question. But after some thinking, I decided to not import them, but also start from scatch.

I only changed the following settings:

* Account Settings > Server Settings > Server Settings

  * Check for new messages at startup
  * Check for new messages every 90 minutes

```
user_pref("mail.server.server1.check_new_mail", false);
user_pref("mail.server.server1.check_time", 90);
```

* Account Setting > Server Settings > Junk Settings

  * When new junk messages to > "Junk" folder on dpat@syn-net.org

* Thunderbird Settings > General > Default Search Engine: DuckDuckGo
* Thunderbird Settings > Privacy & Security > Junk

  * When I mark messages as junk > Move them to the accounts "Junk" folder
  * Mark messages determeinded to be Junk as read
  * Enable adaptive junk filter loggin

* How to install Thunderbird Extensions / Add-ons automatically?

### Thunderbird: Certificate Exceptions

The TLS Certificate Exceptions are saved in `cert9.db` and `cert_override.txt`.
You could review and copy them or rebuild them from scratch.

FTR, connected to an "untrusted" network (ie. a network not controlled by myself) where a Sophos firewall with "Deep Paket Inspection" was installed.
I think I accidentally accepted the new certificate "permanently" (but can not tell for sure anymore).
After leaving that network I received the following error when connecting to my IMAP server:

> "Non-overridable TLS error occurred. Handshake error or probably the TLS version or certificate used by the server imap.syn-net.org is incompatible."

I tried to remove and re-add the certificate (in Settings > Privacy & Security > Manage Certificates) but got the error:

!["No Information Available". Unable to obtain identification status for this site.](screenshot_20231020T094904.png)

I fixed it by deleting `cert9.db` + `cert_override.txt`.

### Thunderbird: Addressbook

Noticed a lot of abook sqlite files:

```
❯ l /mnt/jkirk/.thunderbird/bbq2wowx.default/abook*
-rw-r--r-- 1 jkirk jkirk  66029 Oct  6  2020 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook.mab.bak
-rw-r--r-- 1 jkirk jkirk 458752 Oct 13 20:45 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook.sqlite
-rw-r--r-- 1 jkirk jkirk 458752 Jan  5  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook.v2.sqlite
-rw-r--r-- 1 jkirk jkirk 458752 Oct  8  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook.v3.sqlite
-rw-r--r-- 1 jkirk jkirk  54214 Sep 29  2020 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-1.mab.bak
-rw-r--r-- 1 jkirk jkirk 393216 Oct  8  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-1.sqlite
-rw-r--r-- 1 jkirk jkirk 393216 Jan  5  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-1.v2.sqlite
-rw-r--r-- 1 jkirk jkirk 393216 Oct  8  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-1.v3.sqlite
-rw-r--r-- 1 jkirk jkirk   1438 Oct 28  2009 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-1.mab_
-rw-r--r-- 1 jkirk jkirk   1611 Dec  5  2019 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-2.mab.bak
-rw-r--r-- 1 jkirk jkirk 327680 Oct  8  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-2.sqlite
-rw-r--r-- 1 jkirk jkirk 327680 Jan  5  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-2.v2.sqlite
-rw-r--r-- 1 jkirk jkirk 327680 Oct  8  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-2.v3.sqlite
-rw-r--r-- 1 jkirk jkirk 327680 Oct  8  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-3.sqlite
-rw-r--r-- 1 jkirk jkirk 327680 Jan  5  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-3.v2.sqlite
-rw-r--r-- 1 jkirk jkirk 327680 Oct  8  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-3.v3.sqlite
-rw-r--r-- 1 jkirk jkirk 327680 Dec 15  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-4.sqlite
-rw-r--r-- 1 jkirk jkirk 327680 Jan  5  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-4.v2.sqlite
-rw-r--r-- 1 jkirk jkirk 327680 Oct  8  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-4.v3.sqlite
-rw-r--r-- 1 jkirk jkirk 262144 Jun 11 20:20 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-5.sqlite
-rw-r--r-- 1 jkirk jkirk  67146 Nov  3  2009 /mnt/jkirk/.thunderbird/bbq2wowx.default/abook.mab_


❯ l /mnt/jkirk/.thunderbird/bbq2wowx.default/impab-[^\.].sqlite /mnt/jkirk/.thunderbird/bbq2wowx.default/impab.sqlite
-rw-r--r-- 1 jkirk jkirk  819200 Oct  7 10:59 /mnt/jkirk/.thunderbird/bbq2wowx.default/impab.sqlite
-rw-r--r-- 1 jkirk jkirk 1081344 Sep 24 18:18 /mnt/jkirk/.thunderbird/bbq2wowx.default/impab-1.sqlite
-rw-r--r-- 1 jkirk jkirk  393216 Sep 24 18:18 /mnt/jkirk/.thunderbird/bbq2wowx.default/impab-2.sqlite
-rw-r--r-- 1 jkirk jkirk  327680 Oct  8  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/impab-3.sqlite
-rw-r--r-- 1 jkirk jkirk  327680 Oct  8  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/impab-4.sqlite
-rw-r--r-- 1 jkirk jkirk 1441792 Oct  7 10:59 /mnt/jkirk/.thunderbird/bbq2wowx.default/impab-5.sqlite

❯ l /mnt/jkirk/.thunderbird/bbq2wowx.default/history.*
-rw-r--r-- 1 jkirk jkirk  299285 Oct  7  2020 /mnt/jkirk/.thunderbird/bbq2wowx.default/history.mab.bak
-rw-r--r-- 1 jkirk jkirk 1966080 Oct 13 20:45 /mnt/jkirk/.thunderbird/bbq2wowx.default/history.sqlite
-rw-r--r-- 1 jkirk jkirk 1835008 Jan  5  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/history.v2.sqlite
-rw-r--r-- 1 jkirk jkirk 1835008 Oct  8  2022 /mnt/jkirk/.thunderbird/bbq2wowx.default/history.v3.sqlite
-rw-r--r-- 1 jkirk jkirk    7920 Feb 12  2008 /mnt/jkirk/.thunderbird/bbq2wowx.default/history.mab_
```

I omitted the `v2` and `v3` files:

```sh
❯ cp /mnt/jkirk/.thunderbird/bbq2wowx.default/history.sqlite .thunderbird/gi77x3jn.default-default/history.sqlite

❯ cp -a /mnt/jkirk/.thunderbird/bbq2wowx.default/abook-[^\.].sqlite .thunderbird/gi77x3jn.default-default/

❯ cp -a /mnt/jkirk/.thunderbird/bbq2wowx.default/impab-[^\.].sqlite /mnt/jkirk/.thunderbird/bbq2wowx.default/impab.sqlite .thunderbird/gi77x3jn.default-default
```

Review: prefs.js

Something like this should exist for each file:
```
      user_pref("ldap_2.servers.Lsungsweg.description", "Lösungsweg");
      user_pref("ldap_2.servers.Lsungsweg.dirType", 101);
      user_pref("ldap_2.servers.Lsungsweg.filename", "abook-3.sqlite");
      user_pref("ldap_2.servers.Lsungsweg.uid", "f66bbc4e-eb3f-4c9f-9ff9-309986086a2e");
```

### Thunderbird: Filter

```sh
❯ cat .thunderbird/gi77x3jn.default-default/ImapMail/imap.syn-net.org/msgFilterRules.dat
version="9"
logging="no"

❯ cp /mnt/jkirk/.thunderbird/bbq2wowx.default/ImapMail/syn-net.org/msgFilterRules.dat .thunderbird/gi77x3jn.default-default/ImapMail/imap.syn-net.org/msgFilterRules.dat
```

Had to create custom headers: `x-debian-pr-package`

### Thunderbird: End-To-End Encryption

Checked my private keys. Some of them are obsolete, so took the one I usually use:

```sh
  ❯ gpg -K
  /home/jkirk/.gnupg/pubring.kbx
  ------------------------------
  [...]
  sec   rsa4096 2015-01-29 [SC] [expires: 2026-02-13]
        8DF2271871E57DA5714C8EDE9BB6983DDD81AFEB
  uid           [ultimate] Darshaka Pathirana <dpat@syn-net.org>
  uid           [ultimate] Darshaka Pathirana <d@synpro.solutions>
  uid           [ultimate] Darshaka Pathirana <darshaka.pathirana@synaptic-networks.com>
  uid           [ultimate] Darshaka Pathirana <darshaka.pathirana@synpro.solutions>
  ssb   rsa4096 2015-01-29 [E] [expires: 2026-02-13]
  [...]

  ❯ gpg --export-secret-keys --armor 8DF2271871E57DA5714C8EDE9BB6983DDD81AFEB > my-secret-keys.asc
```

* Account Setting > End-To-End Encryption

  Add Key > Import an existing OpenPGP Key > selected `my-secret-keys.asc`

  Checked "Sign unencrypted messages"

  Unchecked "Attach my public key when adding an OpenPGP digital signature"

```sh
❯ gpg --export --armor > all-public-keys.asc
```

![This file is too big](screenshot_20231015T013620.png)

Quoting [OpenPGP in Thunderbird - HOWTO and FAQ](https://support.mozilla.org/en-US/kb/openpgp-thunderbird-howto-and-faq):

> However, if you have many keys, you might experience a problem because of a current limitation in Thunderbird.
> Currently, Thunderbird cannot import a large set of keys in a single step. An attempt to import a file that is bigger than 5 MB will be rejected.
>
> You have two options to work around this limitation.
>
> The first option is to use a graphical key manager for GnuPG and export your
> keys into separate files. For example, if all public keys in total have a
> size of 17 MB, you would have to create 4 files, and select a quarter of
> public keys for each exported file. This is a bit cumbersome.
>
> Alternatively, you could try to use the Enigmail version 2.2.x migration
> Add-on for importing public keys into Thunderbird, even if you haven't used
> Enigmail before.
>
> [...]

I didn't want to use a "graphical key manager for GnuPG" (which one?).

So I counted the number of public keys in my keyring and created 2 public key files:

```sh
❯ gpg --list-keys --with-colons | grep pub | wc -l
70

❯ gpg --list-keys --with-colons | grep pub | cut -f 5 -d : | head -30 | xargs gpg --export --armor > all-public-keys-1.asc
❯ gpg --list-keys --with-colons | grep pub | cut -f 5 -d : | tail -40 | xargs gpg --export --armor > all-public-keys-2.asc
❯ ls -l all-public-keys-*
-rw-r--r-- 1 jkirk jkirk 3588165 Oct 15 02:23 all-public-keys-1.asc
-rw-r--r-- 1 jkirk jkirk 3973623 Oct 15 02:23 all-public-keys-2.asc
```

Do you accept these keys for verifiying digital signatures and for encrypting
messages, for all shown email addresses?

- Not accepted (undecided)
- Accepted (unverified)

![Accept the key for verififying digital signatures and for ecnrypting messages?](screenshot_20231015T022519.png)

Selecting "Not accepted (undecided) leads to the following "Acceptance":

* Not yet, maybe later

![Key Properties: Not accepted (undecided)](screenshot_20231015T023734.png)

Selecting "Accepted (unverified)" leads to the following "Acceptance":

* Yes, but I have not verified that this is the correct key

I also tried Mikas key, which I have verified and signed a while ago:

```sh
❯ gpg --edit-key 33CCB136401AFEC843A3876396A87872B7EA3737
gpg (GnuPG) 2.2.40; Copyright (C) 2022 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


pub  rsa4096/96A87872B7EA3737
     created: 2010-07-14  expires: 2025-10-01  usage: SC
     trust: full          validity: unknown
sub  rsa4096/257B6FD52892CF7E
     created: 2010-07-14  expires: never       usage: E
sub  rsa4096/43A1FB9BCC87C963
     created: 2018-07-26  expires: 2025-10-01  usage: A
[ unknown] (1). Michael Prokop <mail@michael-prokop.at>
[ unknown] (2)  Michael Prokop <mika@grml.org>
[ unknown] (3)  Michael Prokop <mika@debian.org>
[ revoked] (4)  Michael Prokop <michael@linuxtage.at>
[ unknown] (5)  Michael Prokop <prokop@grml-solutions.com>
[ unknown] (6)  Michael Prokop <michael.prokop@synpro.solutions>

❯ gpg --check-sigs 33CCB136401AFEC843A3876396A87872B7EA3737 | grep dpat
sig!1        9BB6983DDD81AFEB 2015-10-19  Darshaka Pathirana <dpat@syn-net.org>
sig!1        9BB6983DDD81AFEB 2015-10-19  Darshaka Pathirana <dpat@syn-net.org>
sig!1        9BB6983DDD81AFEB 2015-10-19  Darshaka Pathirana <dpat@syn-net.org>
sig!1        9BB6983DDD81AFEB 2015-10-19  Darshaka Pathirana <dpat@syn-net.org>
sig!3 L      9BB6983DDD81AFEB 2015-10-19  Darshaka Pathirana <dpat@syn-net.org>
gpg: 98 good signatures
gpg: 250 signatures not checked due to missing keys
```

To Thunderbird importer does not take into account that I have already verfied the key.

I decided to not invest any further energy on this.
The Thunderbird OpenGPG key ring will only be my secondary store, so I will set all imported keys to "Accepted (unverified)" until I notice problems.

See:

* https://support.mozilla.org/en-US/kb/thunderbird-help-setup-account-e2ee#w_your-own-openpgp-configuration
* https://support.mozilla.org/en-US/kb/openpgp-thunderbird-howto-and-faq#w_what-does-key-acceptance-mean
* help understanding gpg --list--keys output - Unix & Linux Stack Exchange: https://unix.stackexchange.com/questions/613839/help-understanding-gpg-list-keys-output

TODO: primary password

* https://support.mozilla.org/en-US/kb/protect-your-thunderbird-passwords-primary-password

### Thunderbird: Add-Ons + Settings

Thunderbird-Addons settings are saved in the storage system based on the Web Storage API: https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/storage
I could not figure out how to export that data and for some Addons I could not even find the local storage.

* Check and Send :: Add-ons for Thunderbird: https://addons.thunderbird.net/en-US/thunderbird/addon/check-and-send/

  Why: I like some have some checks before I send my mails. The defaults are fine.

* Copy Message ID :: Add-ons for Thunderbird: https://addons.thunderbird.net/en-US/thunderbird/addon/copy-message-id/

  Adds a button to the message view toolbar to copy the message ID to the clipboard.

* Correct Identity :: Add-ons for Thunderbird: https://addons.thunderbird.net/en-US/thunderbird/addon/correct-identity/

  Why: TODO

* External Editor Revived :: Add-ons for Thunderbird: https://addons.thunderbird.net/en-US/thunderbird/addon/external-editor-revived/ (+ messaging host: https://github.com/Frederick888/external-editor-revived/wiki/Linux, https://github.com/Frederick888/external-editor-revived/releases)

  Why: I like to use gVim as my external editor for my emails.

  Set gVim as external editor and set up the messaging host.

```
user_pref("mail.wrap_long_lines", false);
user_pref("mailnews.send_plaintext_flowed", false);
user_pref("mailnews.wraplength", 0);
```

* Header Tools Improved :: Add-ons for Thunderbird: https://addons.thunderbird.net/en-US/thunderbird/addon/header-tools-improved/

  Why: Allows to modify headers and source of messages. I sometimes need to fix "In-Reply" message header.
* LookOut (fix version) :: Add-ons for Thunderbird: https://addons.thunderbird.net/en-US/thunderbird/addon/lookout-fix-version/

  Why: LookOut decodes winmail.dat (TNEF encoded) files that may come from a misconfigured Microsoft Exchange server or Outlook user allowing access to the original attachments

* Simple Mail Redirection :: Add-ons for Thunderbird: https://addons.thunderbird.net/en-US/thunderbird/addon/simple-mail-redirection/

  Why: Allows me to bounce / redirect emails.

* ToggleReplied :: Add-ons for Thunderbird: https://addons.thunderbird.net/en-US/thunderbird/addon/togglereplied-2/

  Why: I sometimes need to toggle the reply state.

* tbkeys-lite :: Add-ons for Thunderbird: https://addons.thunderbird.net/en-US/thunderbird/addon/tbkeys-lite/

  Why: I am used to Vim keys and like to use j/k to move to the next/previous message. To mark a junk mail I use capital "J" and added the following line:

  ::

        "J": "cmd:cmd_markAsJunk",

(curretly) not compatible with Thunderbird 115:

* Nostalgy++/ Manage, search and archive emails :: Add-ons for Thunderbird: https://addons.thunderbird.net/en-US/thunderbird/addon/nostalgy_ng/

I needed to find out how to install add-ons using a script, but there was little to find.
This was the most promissing, I found: https://stackoverflow.com/questions/38469757/programatically-install-add-on-supporting-automatic-updates

My proof-of-concept script looked like this:
I used `https://addons.thunderbird.net/thunderbird/downloads/latest/$add-on-name` to download the latest version of the Thunderbird Add-On.
XPI is just a zip file and inside the zip file there is a manifest.json with its ID either in `.applications.gecko.id` or `.browser_specific_settings.gecko.id` (as learned later).

```sh
❯ wget -O tmp.xpi https://addons.thunderbird.net/thunderbird/downloads/latest/simple-mail-redirection/
❯ ID=$(unzip -p tmp.xpi manifest.json | grep -v "^.*//" | jq -r .applications.gecko.id)
❯ mv tmp.xpi .thunderbird/gi77x3jn.default-default/extensions/$ID.xpi
```

```sh
❯ cat << 'EOF' | while read p; do echo $p; wget -q -O tmp.xpi "https://addons.thunderbird.net/thunderbird/downloads/latest/$p"; ID=$(unzip -p tmp.xpi manifest.json | grep -v "^.*//" | jq -r .applications.gecko.id); echo $ID; mv tmp.xpi ".thunderbird/gi77x3jn.default-default/extensions/${ID}.xpi"; done
check-and-send
correct-identity
external-editor-revived
lookout-fix-version
simple-mail-redirection
EOF

❯ cat << 'EOF' | while read p; do echo $p; wget -q -O tmp.xpi "https://addons.thunderbird.net/thunderbird/downloads/latest/$p"; ID=$(unzip -p tmp.xpi manifest.json | grep -v "^.*//" | jq -r .browser_specific_settings.gecko.id); echo $ID; mv tmp.xpi ".thunderbird/gi77x3jn.default-default/extensions/${ID}.xpi"; done
tbkeys-lite
togglereplied-2
EOF
```

```sh
❯ l .thunderbird/gi77x3jn.default-default/extensions
total 320
-rw-r--r-- 1 jkirk jkirk 10474 Oct 12 17:12 copy-message-id@j.kahn.xpi
-rw-r--r-- 1 jkirk jkirk 76887 Feb 10  2023 external-editor-revived@tsundere.moe.xpi
-rw-r--r-- 1 jkirk jkirk 47572 Jul 20 03:20 lookout@s3_fix_version.xpi
-rw-r--r-- 1 jkirk jkirk 84619 Aug 16 15:36 simplemailredirection@ggbs.de.xpi
drwxr-xr-x 3 jkirk jkirk  4096 Oct 12 17:06 staged
-rw-r--r-- 1 jkirk jkirk 22492 Jul 20 16:36 tbkeys-lite@addons.thunderbird.net.xpi
-rw-r--r-- 1 jkirk jkirk  9715 Aug 12 19:35 togglereplied@kamens.us.xpi
-rw-r--r-- 1 jkirk jkirk 30333 Feb 18  2023 {1B0ADFEC-846C-401D-BA54-7842CBD485D4}.xpi
-rw-r--r-- 1 jkirk jkirk 28377 Jun 19 21:42 {47ef7cc0-2201-11da-8cd6-0800200c9a66}.xpi
```

To enable external editor reviewed, the native messaging host needs to be installed: https://github.com/Frederick888/external-editor-revived/wiki/Linux

Downloded the lastest `ubuntu-latest-gnu-native-messaging-host-vX.Y.Z.zip` to my scripts directory and run

```sh
❯ external-editor-revived | tee "$HOME/.mozilla/native-messaging-hosts/external_editor_revived.json"
Please create 'external_editor_revived.json' manifest file with the JSON below.
Consult https://wiki.mozilla.org/WebExtensions/Native_Messaging for its location.

{
  "name": "external_editor_revived",
  "description": "Edit emails in external editors such as Vim, Neovim, Emacs, etc.",
  "path": "/home/jkirk/projects/scripts/external-editor-revived",
  "type": "stdio",
  "allowed_extensions": [
    "external-editor-revived@tsundere.moe"
  ]
}
```

### gnote

```sh
❯ cp -a /mnt/jkirk/.config/gnote .config
❯ cp -a /mnt/jkirk/.local/share/gnote .local/share/
```

### Time Zone

Somehow the time (zone) changed after I suspended the the system.
The system time was off 2 hours after I woke up my system.

```
Oct 15 19:50:42 tranquility systemd-logind[1066]: The system will suspend now!
Oct 15 19:50:42 tranquility NetworkManager[1090]: <info>  [1697392242.5012] manager: sleep: sleep requested (sleeping: no  enabled: yes)
Oct 15 19:50:42 tranquility NetworkManager[1090]: <info>  [1697392242.5013] device (p2p-dev-wlp0s20f3): state change: disconnected -> unmanaged (reason 'sleeping', sys-iface-state: 'managed')
Oct 15 19:50:42 tranquility ModemManager[1116]: <info>  [sleep-monitor-systemd] system is about to suspend
Oct 15 19:50:42 tranquility NetworkManager[1090]: <info>  [1697392242.5019] device (wwan0mbim0): state change: disconnected -> unmanaged (reason 'sleeping', sys-iface-state: 'managed')
Oct 15 19:50:42 tranquility NetworkManager[1090]: <info>  [1697392242.5022] manager: NetworkManager state is now ASLEEP
Oct 15 19:50:42 tranquility NetworkManager[1090]: <info>  [1697392242.5024] device (wlp0s20f3): state change: activated -> deactivating (reason 'sleeping', sys-iface-state: 'managed')
Oct 15 19:50:42 tranquility dbus-daemon[1061]: [system] Activating via systemd: service name='org.freedesktop.nm_dispatcher' unit='dbus-org.freedesktop.nm-dispatcher.service' requested by ':1.11' (uid=0 pid=1090 comm="/usr/sbin/NetworkManager --no-daemon")
Oct 15 19:50:42 tranquility systemd[1]: Starting NetworkManager-dispatcher.service - Network Manager Script Dispatcher Service...
Oct 15 19:50:42 tranquility dbus-daemon[1061]: [system] Successfully activated service 'org.freedesktop.nm_dispatcher'
Oct 15 19:50:42 tranquility systemd[1]: Started NetworkManager-dispatcher.service - Network Manager Script Dispatcher Service.
Oct 15 19:50:42 tranquility kernel: wlp0s20f3: deauthenticating from 36:e5:06:bd:99:85 by local choice (Reason: 3=DEAUTH_LEAVING)
Oct 15 19:50:42 tranquility wpa_supplicant[1094]: wlp0s20f3: CTRL-EVENT-DISCONNECTED bssid=36:e5:06:bd:99:85 reason=3 locally_generated=1
Oct 15 19:50:42 tranquility wpa_supplicant[1094]: wlp0s20f3: CTRL-EVENT-DSCP-POLICY clear_all
Oct 15 19:50:42 tranquility NetworkManager[1090]: <info>  [1697392242.6825] device (wlp0s20f3): supplicant interface state: completed -> disconnected
Oct 15 19:50:42 tranquility NetworkManager[1090]: <info>  [1697392242.6829] device (wlp0s20f3): state change: deactivating -> disconnected (reason 'sleeping', sys-iface-state: 'managed')
Oct 15 19:50:42 tranquility avahi-daemon[1052]: Withdrawing address record for fe80::bdc:8647:82c1:144e on wlp0s20f3.
Oct 15 19:50:42 tranquility avahi-daemon[1052]: Leaving mDNS multicast group on interface wlp0s20f3.IPv6 with address fe80::bdc:8647:82c1:144e.
Oct 15 19:50:42 tranquility avahi-daemon[1052]: Interface wlp0s20f3.IPv6 no longer relevant for mDNS.
Oct 15 19:50:42 tranquility NetworkManager[1090]: <info>  [1697392242.7128] dhcp4 (wlp0s20f3): canceled DHCP transaction
Oct 15 19:50:42 tranquility NetworkManager[1090]: <info>  [1697392242.7129] dhcp4 (wlp0s20f3): activation: beginning transaction (timeout in 45 seconds)
Oct 15 19:50:42 tranquility NetworkManager[1090]: <info>  [1697392242.7129] dhcp4 (wlp0s20f3): state changed no lease
Oct 15 19:50:42 tranquility avahi-daemon[1052]: Interface wlp0s20f3.IPv4 no longer relevant for mDNS.
Oct 15 19:50:42 tranquility avahi-daemon[1052]: Leaving mDNS multicast group on interface wlp0s20f3.IPv4 with address 192.168.43.56.
Oct 15 19:50:42 tranquility avahi-daemon[1052]: Withdrawing address record for 192.168.43.56 on wlp0s20f3.
Oct 15 19:50:42 tranquility NetworkManager[1090]: <info>  [1697392242.7497] device (wlp0s20f3): set-hw-addr: set MAC address to D6:3D:19:16:AA:FE (scanning)
Oct 15 19:50:42 tranquility avahi-daemon[1052]: Joining mDNS multicast group on interface wlp0s20f3.IPv4 with address 192.168.43.56.
Oct 15 19:50:42 tranquility avahi-daemon[1052]: New relevant interface wlp0s20f3.IPv4 for mDNS.
Oct 15 19:50:42 tranquility avahi-daemon[1052]: Registering new address record for 192.168.43.56 on wlp0s20f3.IPv4.
Oct 15 19:50:42 tranquility avahi-daemon[1052]: Withdrawing address record for 192.168.43.56 on wlp0s20f3.
Oct 15 19:50:42 tranquility avahi-daemon[1052]: Leaving mDNS multicast group on interface wlp0s20f3.IPv4 with address 192.168.43.56.
Oct 15 19:50:42 tranquility avahi-daemon[1052]: Interface wlp0s20f3.IPv4 no longer relevant for mDNS.
Oct 15 19:50:42 tranquility NetworkManager[1090]: <info>  [1697392242.8293] device (wlp0s20f3): state change: disconnected -> unmanaged (reason 'sleeping', sys-iface-state: 'managed')
Oct 15 19:50:43 tranquility NetworkManager[1090]: <info>  [1697392243.0708] device (wlp0s20f3): set-hw-addr: reset MAC address to F4:3B:D8:A9:F0:97 (unmanage)
Oct 15 19:50:43 tranquility systemd[1]: Reached target sleep.target - Sleep.
Oct 15 19:50:43 tranquility systemd[1]: Starting syncthing-resume.service - Restart Syncthing after resume...
Oct 15 19:50:43 tranquility systemd[1]: Starting systemd-suspend.service - System Suspend...
Oct 15 19:50:43 tranquility wpa_supplicant[1094]: p2p-dev-wlp0s20: CTRL-EVENT-DSCP-POLICY clear_all
Oct 15 19:50:43 tranquility wpa_supplicant[1094]: p2p-dev-wlp0s20: CTRL-EVENT-DSCP-POLICY clear_all
Oct 15 19:50:43 tranquility wpa_supplicant[1094]: nl80211: deinit ifname=p2p-dev-wlp0s20 disabled_11b_rates=0
Oct 15 19:50:43 tranquility systemd-sleep[64469]: Entering sleep state 'suspend'...
Oct 15 19:50:43 tranquility kernel: PM: suspend entry (s2idle)
Oct 15 19:50:43 tranquility kernel: Filesystems sync: 0.008 seconds
Oct 15 19:50:43 tranquility kernel: (NULL device *): firmware: direct-loading firmware i915/adlp_dmc_ver2_16.bin
Oct 15 19:50:43 tranquility kernel: (NULL device *): firmware: direct-loading firmware regulatory.db
Oct 15 19:50:43 tranquility kernel: (NULL device *): firmware: direct-loading firmware i915/adlp_guc_70.bin
Oct 15 19:50:43 tranquility kernel: (NULL device *): firmware: direct-loading firmware iwlwifi-so-a0-gf-a0.pnvm
Oct 15 19:50:43 tranquility kernel: (NULL device *): firmware: direct-loading firmware intel/ibt-0040-0041.ddc
Oct 15 19:50:43 tranquility kernel: (NULL device *): firmware: direct-loading firmware regulatory.db.p7s
Oct 15 19:50:43 tranquility kernel: (NULL device *): firmware: direct-loading firmware intel/sof-tplg/sof-hda-generic-4ch.tplg
Oct 15 19:50:43 tranquility kernel: (NULL device *): firmware: direct-loading firmware i915/tgl_huc.bin
Oct 15 22:47:12 tranquility kernel: (NULL device *): firmware: direct-loading firmware intel/ibt-0040-0041.sfi
Oct 15 22:47:12 tranquility kernel: (NULL device *): firmware: direct-loading firmware iwlwifi-so-a0-gf-a0-72.ucode
Oct 15 22:47:12 tranquility kernel: Freezing user space processes
Oct 15 22:47:12 tranquility kernel: Freezing user space processes completed (elapsed 0.031 seconds)
Oct 15 22:47:12 tranquility kernel: OOM killer disabled.
Oct 15 22:47:12 tranquility kernel: Freezing remaining freezable tasks
Oct 15 22:47:12 tranquility kernel: Freezing remaining freezable tasks completed (elapsed 0.002 seconds)
Oct 15 22:47:12 tranquility kernel: printk: Suspending console(s) (use no_console_suspend to debug)
Oct 15 22:47:12 tranquility kernel: ACPI: EC: interrupt blocked
Oct 15 22:47:12 tranquility kernel: typec port1-partner: PM: parent port1 should not be sleeping
Oct 15 22:47:12 tranquility kernel: ACPI: EC: interrupt unblocked
Oct 15 22:47:12 tranquility kernel: i915 0000:00:02.0: [drm] GuC firmware i915/adlp_guc_70.bin version 70.5.1
Oct 15 22:47:12 tranquility kernel: i915 0000:00:02.0: [drm] HuC firmware i915/tgl_huc.bin version 7.9.3
Oct 15 22:47:12 tranquility kernel: nvme nvme0: Shutdown timeout set to 10 seconds
Oct 15 22:47:12 tranquility kernel: nvme nvme0: 12/0/0 default/read/poll queues
Oct 15 22:47:12 tranquility kernel: i915 0000:00:02.0: [drm] HuC authenticated
Oct 15 22:47:12 tranquility kernel: i915 0000:00:02.0: [drm] GuC submission enabled
Oct 15 22:47:12 tranquility kernel: i915 0000:00:02.0: [drm] GuC SLPC enabled
Oct 15 22:47:12 tranquility kernel: i915 0000:00:02.0: [drm] GuC RC: enabled
Oct 15 22:47:12 tranquility kernel: thinkpad_acpi: undocked from hotplug port replicator
Oct 15 22:47:12 tranquility kernel: mei_hdcp 0000:00:16.0-b638ab7e-94e2-4ea2-a552-d1c54b627f04: bound 0000:00:02.0 (ops i915_hdcp_component_ops [i915])
Oct 15 22:47:12 tranquility kernel: OOM killer enabled.
Oct 15 22:47:12 tranquility kernel: Restarting tasks ...
Oct 15 22:47:12 tranquility kernel: usb 1-6: USB disconnect, device number 15
Oct 15 22:47:12 tranquility wpa_supplicant[1094]: wlp0s20f3: CTRL-EVENT-DSCP-POLICY clear_all
Oct 15 22:47:12 tranquility kernel: done.
Oct 15 22:47:12 tranquility kernel: random: crng reseeded on system resumption
Oct 15 22:47:12 tranquility systemd[1]: anacron.service - Run anacron jobs was skipped because of an unmet condition check (ConditionACPower=true).
Oct 15 22:47:12 tranquility systemd[1]: syncthing-resume.service: Deactivated successfully.
Oct 15 22:47:12 tranquility systemd[1]: Finished syncthing-resume.service - Restart Syncthing after resume.
Oct 15 22:47:12 tranquility wpa_supplicant[1094]: wlp0s20f3: CTRL-EVENT-DSCP-POLICY clear_all
Oct 15 22:47:12 tranquility wpa_supplicant[1094]: nl80211: deinit ifname=wlp0s20f3 disabled_11b_rates=0
Oct 15 22:47:12 tranquility kernel: usb 1-6: new full-speed USB device number 16 using xhci_hcd
Oct 15 22:47:12 tranquility dbus-daemon[1061]: [system] Activating via systemd: service name='org.freedesktop.PackageKit' unit='packagekit.service' requested by ':1.72' (uid=1000 pid=1862 comm="/usr/bin/gnome-software --gapplication-service")
Oct 15 22:47:12 tranquility systemd-sleep[64469]: System returned from sleep state.
Oct 15 22:47:12 tranquility kernel: PM: suspend exit
Oct 15 22:47:12 tranquility bluetoothd[1054]: Controller resume with wake event 0x0
Oct 15 22:47:12 tranquility systemd[1]: Starting packagekit.service - PackageKit Daemon...
Oct 15 22:47:12 tranquility systemd[1]: systemd-suspend.service: Deactivated successfully.
Oct 15 22:47:12 tranquility systemd[1]: Finished systemd-suspend.service - System Suspend.
Oct 15 22:47:12 tranquility systemd[1]: Stopped target sleep.target - Sleep.
Oct 15 22:47:12 tranquility systemd[1]: Reached target suspend.target - Suspend.
Oct 15 22:47:12 tranquility systemd[1]: Stopped target suspend.target - Suspend.
```

Note, that I connect my notebook to the Thunderbolt Docking Station for the first time and turned it off at about 20:47 CEST.

I checked the time zone setting:

```sh
❯ timedatectl
               Local time: Sun 2023-10-15 23:04:43 CEST
           Universal time: Sun 2023-10-15 21:04:43 UTC
                 RTC time: Sun 2023-10-15 21:04:43
                Time zone: Europe/Vienna (CEST, +0200)
System clock synchronized: no
              NTP service: active
          RTC in local TZ: yes

Warning: The system is configured to read the RTC time in the local time zone.
         This mode cannot be fully supported. It will create various problems
         with time zone changes and daylight saving time adjustments. The RTC
         time is never updated, it relies on external facilities to maintain it.
         If at all possible, use RTC in UTC by calling
         'timedatectl set-local-rtc 0'.
```

and fixed it with:

```sh
❯ timedatectl set-local-rtc 1 --adjust-system-clock
❯ timedatectl
               Local time: Sun 2023-10-15 21:08:37 CEST
           Universal time: Sun 2023-10-15 19:08:37 UTC
                 RTC time: Sun 2023-10-15 21:08:37
                Time zone: Europe/Vienna (CEST, +0200)
System clock synchronized: no
              NTP service: active
          RTC in local TZ: yes

Warning: The system is configured to read the RTC time in the local time zone.
         This mode cannot be fully supported. It will create various problems
         with time zone changes and daylight saving time adjustments. The RTC
         time is never updated, it relies on external facilities to maintain it.
         If at all possible, use RTC in UTC by calling
         'timedatectl set-local-rtc 0'.
```

I could not figure why this happened, systemd-timesyncd did not log anything special:

```sh
❯ sudo journalctl -u systemd-timesyncd.service --boot
Oct 12 10:53:23 tranquility systemd[1]: Starting systemd-timesyncd.service - Network Time Synchronization...
Oct 12 10:53:23 tranquility systemd-timesyncd[1018]: The system is configured to read the RTC time in the local time zone. This mode cannot be fully supported. All system time to RTC updates are disabled.
Oct 12 10:53:23 tranquility systemd[1]: Started systemd-timesyncd.service - Network Time Synchronization.
Oct 12 10:53:48 tranquility systemd-timesyncd[1018]: Contacted time server 162.159.200.1:123 (2.debian.pool.ntp.org).
Oct 12 10:53:48 tranquility systemd-timesyncd[1018]: Initial clock synchronization to Thu 2023-10-12 10:53:48.853215 CEST.
Oct 12 16:55:42 tranquility systemd-timesyncd[1018]: Contacted time server 185.119.117.217:123 (0.debian.pool.ntp.org).
Oct 13 21:19:42 tranquility systemd-timesyncd[1018]: Contacted time server 37.252.188.90:123 (0.debian.pool.ntp.org).
Oct 14 00:05:47 tranquility systemd-timesyncd[1018]: Contacted time server 144.76.197.108:123 (0.debian.pool.ntp.org).
Oct 14 15:15:58 tranquility systemd-timesyncd[1018]: Contacted time server 91.206.8.34:123 (0.debian.pool.ntp.org).
Oct 14 16:37:10 tranquility systemd-timesyncd[1018]: Contacted time server 131.130.251.107:123 (0.debian.pool.ntp.org).
Oct 15 01:35:37 tranquility systemd-timesyncd[1018]: Contacted time server 162.159.200.123:123 (0.debian.pool.ntp.org).
```

I later changed that to:

```
❯ sudo timedatectl set-local-rtc 0
```

### Thunderbolt

Because of the time zone issue above, I checked my Thunderbolt configuration status.
The device seems have been authorized automatically:

```sh
❯ boltctl
 ● Lenovo ThinkPad Thunderbolt 4 Dock
   ├─ type:          peripheral
   ├─ name:          ThinkPad Thunderbolt 4 Dock
   ├─ vendor:        Lenovo
   ├─ uuid:          d3fb8780-0027-a938-ffff-ffffffffffff
   ├─ generation:    USB4
   ├─ status:        authorized
   │  ├─ domain:     a7b08780-3143-312b-ffff-ffffffffffff
   │  ├─ rx speed:   40 Gb/s = 2 lanes * 20 Gb/s
   │  ├─ tx speed:   40 Gb/s = 2 lanes * 20 Gb/s
   │  └─ authflags:  none
   ├─ authorized:    Sun 15 Oct 2023 08:47:23 PM UTC
   ├─ connected:     Sun 15 Oct 2023 08:47:21 PM UTC
   └─ stored:        Sun 15 Oct 2023 08:47:23 PM UTC
      ├─ policy:     iommu
      └─ key:        no


❯ sudo systemctl status bolt.service
● bolt.service - Thunderbolt system service
     Loaded: loaded (/lib/systemd/system/bolt.service; static)
     Active: active (running) since Thu 2023-10-12 10:53:24 CEST; 3 days ago
       Docs: man:boltd(8)
   Main PID: 1119 (boltd)
     Status: "authmode: enabled, force-power: unset"
      Tasks: 3 (limit: 38041)
     Memory: 1.7M
        CPU: 2.440s
     CGroup: /system.slice/bolt.service
             └─1119 /usr/libexec/boltd

Oct 15 22:47:21 tranquility boltd[1119]: probing: started [1000]
Oct 15 22:47:21 tranquility boltd[1119]: [d3fb8780-0027-ThinkPad Thunderbolt 4 Dock] authorize: authorization prepared for 'user' level
Oct 15 22:47:21 tranquility boltd[1119]: [d3fb8780-0027-ThinkPad Thunderbolt 4 Dock] dbus: exported device at /org/freedesktop/bolt/devices/d3fb8780_0027...
Oct 15 22:47:21 tranquility boltd[1119]: [d3fb8780-0027-ThinkPad Thunderbolt 4 Dock] udev: device changed: authorizing -> authorizing
Oct 15 22:47:21 tranquility boltd[1119]: [d3fb8780-0027-ThinkPad Thunderbolt 4 Dock] udev: device changed: authorizing -> authorizing
Oct 15 22:47:23 tranquility boltd[1119]: [d3fb8780-0027-ThinkPad Thunderbolt 4 Dock] authorize: finished: ok (status: authorized, flags: 0)
Oct 15 22:47:23 tranquility boltd[1119]: [d3fb8780-0027                            ] bootacl: policy not 'auto', not adding
Oct 15 22:47:23 tranquility boltd[1119]: [d3fb8780-0027-ThinkPad Thunderbolt 4 Dock] auto-enroll: done
Oct 15 22:47:23 tranquility boltd[1119]: [d3fb8780-0027-ThinkPad Thunderbolt 4 Dock] udev: device changed: authorized -> authorized
Oct 15 22:47:26 tranquility boltd[1119]: probing: timeout, done: [2788086] (2000000)
```

### Hamster Time Tracking Application

```sh
❯ cp -a /mnt/jkirk/.local/share/hamster/hamster.db .local/share/hamster
```

### Signal Deskop

```sh
❯ cp -a /mnt/jkirk/.config/Signal/ .config
```

First went offline, then started Signal and after everything looked fine, went online.

### Autokey

```sh
❯ cp -a /mnt/jkirk/.config/autokey .config
```

### gnome-screenshot

Better screenshot filename

```
❯ dconf dump /org/cinnamon/desktop/keybindings/custom-keybindings/custom3/
[/]
binding=['<Shift>Print']
command='/home/jkirk/bin/screen-upload.sh -o'
name='screenshot area with better filename'
```

```
❯ dconf dump /org/cinnamon/desktop/keybindings/media-keys/
[/]
area-screenshot=@as []

~
at 2023-10-16 11:21:01 +02:00 ❯ DCONF_PROFILE=/home/jkirk/.config/dconf/profile/executor dconf dump /org/cinnamon/desktop/keybindings/media-keys/
[/]
area-screenshot=@as []
calculator=['XF86Calculator']
next=['XF86AudioNext']
pause=['AudioStop']
play=['AudioPlay']
previous=['XF86AudioPrev']
screensaver=['XF86ScreenSaver', '<Primary><Shift><Alt>l']
search=['XF86Search']
stop=@as []
suspend=['XF86Sleep']
window-screenshot=['<Alt>Print']
```

### Samba Share

I have laser printer/scanner which can scan its documents directly to my notebook.

I created a dedicated user `laserjet`:

```sh
sudo adduser --no-create-home --disabled-password --disabled-login laserjet
sudo smbpasswd -a laserjet
```

And created a dedicated share for the scanner and added the following lines to `/etc/samba/smb.conf`

```
[scan]
   comment = Scan Folder
   browseable = no
   path = /home/jkirk/Documents/scans/laserjet
   guest ok = no
   create mask = 0644
   valid users  = laserjet
   write list = laserjet
   force user = jkirk
   force group = jkirk
```

### virtualbox

Needs linux-headers...

```
❯ sudo lvcreate -L 100G -n vbox vg0-tranquility
  Logical volume "vbox" created.

~
at 2023-10-25 20:10:11 +02:00 ❯ sudo mkfs.ext4 /dev/vg0-tranquility/vbox
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 26214400 4k blocks and 6553600 inodes
Filesystem UUID: 704e7442-ef17-45f1-a854-ef1998f24c1f
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done
Writing inode tables: done
Creating journal (131072 blocks): done
Writing superblocks and filesystem accounting information: done

❯ mkdir "/home/jkirk/VirtualBox VMs"


❯ grep VirtualBox /etc/fstab
/dev/mapper/vg0--tranquility-vbox /home/jkirk/VirtualBox\040VMs          ext4    defaults        0       2

❯ sudo systemctl daemon-reload
❯ sudo mount -a

❯ sudo chown jkirk:jkirk VirtualBox\ VMs

❯ cp -a /mnt/jkirk/.config/VirtualBox .config
```

After installation don't forget:

```
  ❯ sudo adduser jkirk vboxusers 
  [sudo] password for jkirk: 
  Adding user `jkirk' to group `vboxusers' ...
  Done.
```

### docker / podman

```sh
at 2023-10-20 19:56:40 +02:00 ❯ sudo lvcreate -L 20G -n containers vg0-tranquility
  Logical volume "containers" created.

❯ sudo mkfs.ext4 /dev/vg0-tranquility/containers
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 2621440 4k blocks and 655360 inodes
Filesystem UUID: 74779f17-43ca-4673-b62f-459ef73c43ed
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done


at 2023-10-20 19:58:19 +02:00 ❯ mkdir .local/share/containers/



❯ sudo mount /dev/vg0-tranquility/containers /home/jkirk/.local/share/containers

❯ sudo chown jkirk:jkirk /home/jkirk/.local/share/containers
```


```
/dev/mapper/vg0--tranquility-containers /home/jkirk/.local/share/containers           ext4    defaults        0       2
```

```
❯ sudo systemctl daemon-reload
```

#### podman problems

```sh
❯ podman pull wordpress
Error: command required for rootless mode with multiple IDs: exec: "newuidmap": executable file not found in $PATH
```

```
❯ podman pull wordpress
Error: short-name "wordpress" did not resolve to an alias and no unqualified-search registries are defined in "/etc/containers/registries.conf"


❯ podman pull docker.io/wordpress
Trying to pull docker.io/library/wordpress:latest...
Getting image source signatures
Copying blob 12a700ba0368 skipped: already exists
Copying blob 22a19ba793cb skipped: already exists
Copying blob 46e419350351 skipped: already exists
Copying blob 1d7540f99b47 skipped: already exists
Copying blob 9b19829d9fc5 skipped: already exists
Copying blob d44a7be08f84 skipped: already exists
Copying blob 8de997c28946 skipped: already exists
Copying blob 84073d869176 skipped: already exists
Copying blob 0c351c6a1d3f skipped: already exists
Copying blob 6624316b1353 skipped: already exists
Copying blob 81ea0c762078 skipped: already exists
Copying blob 68680de1e42a skipped: already exists
Copying blob 3f6f46332f1b skipped: already exists
Copying blob e67fdae35593 skipped: already exists
Copying blob c42c0b9ab38f skipped: already exists
Copying blob 9d17674da2dd skipped: already exists
Copying blob 91a39b0e19bd skipped: already exists
Copying blob f43f1a40f0fe skipped: already exists
Copying blob 076ed9c917d2 skipped: already exists
Copying blob b6c1fd663115 skipped: already exists
Copying blob a9e97aff2b31 done
Copying config bd918e5d23 done
Writing manifest to image destination
Storing signatures
bd918e5d2324041201a3ec4edb0f61dc9c998773481335128bed98300a3fd4b7

```


Or: https://blog.desigeek.com/post/2022/03/podman-error-on-ubuntu-short-name-did-not-resolve-to-an-alias-and-no-unqualified-search-registries/

```
❯ podman-compose up
[...]
Error: unable to start container f00a3a651a2158423d3acd458592396930871cf26f902d92368a16d503b0d847: failed to mount runtime directory for rootless netns: no such file or directory
exit code: 125
podman start -a wordpress_db_1
Error: unable to start container 4a19bc1a36c32dc73ff71726d975c4773cb780c2e33599557eaa76a9d1c0531b: failed to mount runtime directory for rootless netns: no such file or directory
exit code: 125

❯ podman-compose down
```

```sh
❯ l /run/user/1000/libpod/tmp
total 4
-rw-r--r-- 1 jkirk jkirk  0 Oct 20 19:41 alive
-rw-r--r-- 1 jkirk jkirk  0 Oct 20 19:36 alive.lck
drwxr-x--- 2 jkirk jkirk 40 Oct 21 10:17 exits
-rw------- 1 jkirk jkirk  6 Oct 20 19:41 pause.pid
drwx------ 2 jkirk jkirk 40 Oct 21 10:07 rootless-netns
-rw-r--r-- 1 jkirk jkirk  0 Oct 21 10:07 rootless-netns.lock

l /run/user/1000/netns
total 0
-rw-r--r-- 1 jkirk jkirk 0 Oct 21 10:07 rootless-netns-6ef37e2c05ee6c74c0e2

❯ sudo rm /run/user/1000/netns/rootless-netns-6ef37e2c05ee6c74c0e2

❯ sudo rm /run/user/1000/libpod/tmp/rootless-netns.lock

❯ sudo rm -rf /run/user/1000/libpod/tmp/rootless-netns
```

### Ansible + ara

```
❯ cat /opt/requirements.txt
ara
ara[server]

❯ . /opt/venv3/bin/activate

❯ pip install -r /opt/requirements.txt

❯ cp -a /mnt/jkirk/.ara .
```

```
❯ ara_last
Internal Server Error: /api/v1/playbooks
Traceback (most recent call last):
  File "/opt/venv3/lib/python3.11/site-packages/django/db/backends/utils.py", line 89, in _execute
    return self.cursor.execute(sql, params)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/django/db/backends/sqlite3/base.py", line 328, in execute
    return super().execute(query, params)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
sqlite3.OperationalError: no such column: playbooks.client_version

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/opt/venv3/lib/python3.11/site-packages/django/core/handlers/exception.py", line 55, in inner
    response = get_response(request)
               ^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/django/core/handlers/base.py", line 197, in _get_response
    response = wrapped_callback(request, *callback_args, **callback_kwargs)
               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/django/views/decorators/csrf.py", line 56, in wrapper_view
    return view_func(*args, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/rest_framework/viewsets.py", line 125, in view
    return self.dispatch(request, *args, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/rest_framework/views.py", line 509, in dispatch
    response = self.handle_exception(exc)
               ^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/rest_framework/views.py", line 469, in handle_exception
    self.raise_uncaught_exception(exc)
  File "/opt/venv3/lib/python3.11/site-packages/rest_framework/views.py", line 480, in raise_uncaught_exception
    raise exc
  File "/opt/venv3/lib/python3.11/site-packages/rest_framework/views.py", line 506, in dispatch
    response = handler(request, *args, **kwargs)
               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/rest_framework/mixins.py", line 40, in list
    page = self.paginate_queryset(queryset)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/rest_framework/generics.py", line 171, in paginate_queryset
    return self.paginator.paginate_queryset(queryset, self.request, view=self)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/rest_framework/pagination.py", line 395, in paginate_queryset
    return list(queryset[self.offset:self.offset + self.limit])
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/django/db/models/query.py", line 398, in __iter__
    self._fetch_all()
  File "/opt/venv3/lib/python3.11/site-packages/django/db/models/query.py", line 1881, in _fetch_all
    self._result_cache = list(self._iterable_class(self))
                         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/django/db/models/query.py", line 91, in __iter__
    results = compiler.execute_sql(
              ^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/django/db/models/sql/compiler.py", line 1562, in execute_sql
    cursor.execute(sql, params)
  File "/opt/venv3/lib/python3.11/site-packages/django/db/backends/utils.py", line 67, in execute
    return self._execute_with_wrappers(
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/django/db/backends/utils.py", line 80, in _execute_with_wrappers
    return executor(sql, params, many, context)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/django/db/backends/utils.py", line 84, in _execute
    with self.db.wrap_database_errors:
  File "/opt/venv3/lib/python3.11/site-packages/django/db/utils.py", line 91, in __exit__
    raise dj_exc_value.with_traceback(traceback) from exc_value
  File "/opt/venv3/lib/python3.11/site-packages/django/db/backends/utils.py", line 89, in _execute
    return self.cursor.execute(sql, params)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/venv3/lib/python3.11/site-packages/django/db/backends/sqlite3/base.py", line 328, in execute
    return super().execute(query, params)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
django.db.utils.OperationalError: no such column: playbooks.client_version
2023-11-14 18:53:26,317 ERROR ara.clients.http: Failed to get on /api/v1/playbooks: {'params': {'order': '-started', 'limit': '1'}}
2023-11-14 18:53:26,317 ERROR ara.clients.http: Failed to get on /api/v1/playbooks: {'params': {'order': '-started', 'limit': '1'}}
2023-11-14 18:53:26,317 ERROR ara: Expecting value: line 2 column 1 (char 1)

❯ ara-manage migrate
ara] Using settings file: /home/jkirk/.ara/server/settings.yaml
perations to perform:
 Apply all migrations: admin, api, auth, contenttypes, db, sessions
unning migrations:
 Applying api.0011_play_label_name_max_length... OK
 Applying api.0012_playbook_user... OK
 Applying api.0013_task_status... updated status for 637 task(s) based on failed or unreachable results
OK
 Applying api.0014_ara_versions... OK
 Applying api.0015_task_uuid... OK
 Applying api.0016_revert_play_label_name_length... OK
 Applying api.0017_optional_playbook_controller... OK
 Applying auth.0012_alter_user_first_name_max_length... OK
```

### Ansible + mitogen

```
❯ cd /opt

/opt via 🐍 v3.11.2
at 2023-11-15 10:13:10 +01:00 ❯ git clone https://github.com/dw/mitogen.git

/opt via 🐍 v3.11.2
at 2023-11-15 10:13:10 +01:00 ❯ cd mitogen

/opt/mitogen on  master (798032b) via 🐍 v3.11.2
❯ git log -1
commit 798032b9 (HEAD -> master, origin/master, origin/HEAD)
Merge: d839cbfa 3f105d51
Author: Alex Willmer <alex@moreati.org.uk>
Date:   2023-10-05 14:59:01 +0100

    Merge pull request #1027 from moreati/pyver-token

    ci: Authenticate UsePythonVersion requests to Github
```

```
❯ . /opt/venv3/bin/activate; source <(python3 -m ara.setup.env); export ANSIBLE_STRATEGY=mitogen_linear ANSIBLE_STRATEGY_PLUGINS="/opt/mitogen/ansible_mitogen/plugins/strategy"


❯ ansible-playbook -C -D site.yml
Operations to perform:
  Apply all migrations: admin, api, auth, contenttypes, db, sessions
Running migrations:
  No migrations to apply.

PLAY [Setup site wide monitoring configuration] *******************************************************************************************************************************************************************
ERROR! Your Ansible version ((2, 14, 3)) is too recent. The most recent version
supported by Mitogen for Ansible is (2, 13).x. Please check the Mitogen
release notes to see if a new version is available, otherwise
subscribe to the corresponding GitHub issue to be notified when
support becomes available.

    https://mitogen.rtfd.io/en/latest/changelog.html
    https://github.com/mitogen-hq/mitogen/issues/


❯ ansible --version
ansible [core 2.14.3]
  config file = /home/jkirk/projects/synpro/pmt-ansible/ansible.cfg
  configured module search path = ['/home/jkirk/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/jkirk/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.11.2 (main, Mar 13 2023, 12:18:29) [GCC 12.2.0] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
```


Fixed via

```
/opt/mitogen on  master (798032b) via 🐍 v3.11.2
❯ git remote add moreati https://github.com/moreati/mitogen.git
❯ git fetch moreati
❯ git co -b pr977 moreati/2.14
Switched to branch 'pr977'
Your branch is up to date with 'moreati/2.14'.

/opt/mitogen on  pr977:2.14 (48f6802) via 🐍 v3.11.2

❯ git log -1
commit 48f68025 (HEAD -> pr977, moreati/2.14)
Author: Alex Willmer <alex@moreati.org.uk>
Date:   2023-08-02 18:20:33 +0100

    fixup! Bump ANSIBLE_VERSION_MAX to 2.14
```

Mika mentioned ansible-mitogen which included the simple patch for ansible 2.14 support: https://sources.debian.org/src/python-mitogen/0.3.4-2/debian/patches/ansible-2.14/

So, installing ansible-mitogen is enough.


### Archive

```
❯ sudo du -sch /mnt/Archive
14G     /mnt/Archive
14G     total

~
at 2023-11-04 16:00:08 +01:00 ❯ mkdir Documents/Archive

at 2023-11-04 16:00:49 +01:00 ❯ sudo lvcreate -L 20G -n archive vg0-tranquility
  Logical volume "archive" created.

❯ sudo mkfs.ext4 /dev/vg0-tranquility/archive
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 5242880 4k blocks and 1310720 inodes
Filesystem UUID: 27f41ad0-7a01-448b-aab9-3463d1abb6e3
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

❯ sudo vim /etc/fstab

~ took 6s
at 2023-11-04 16:02:38 +01:00 ❯ sudo systemctl daemon-reload
❯ sudo mount -a
❯ sudo chown jkirk:jkirk Documents/Archive
❯ sudo cp -a /mnt/Archive/. Documents/Archive/

```

### Papersize: A4

Status: unsolved

```
❯ cat /etc/papersize
letter


❯ sudo dpkg-reconfigure libpaper1
[sudo] password for jkirk:
Replacing config file /etc/papersize with new version

❯ cat /etc/papersize
a4
```

See: [How can I ensure that all programs use the same paper size?](https://www.debian.org/doc/manuals/debian-faq/customizing.en.html#papersize)

### LibreOffice

Brauch ich? hunspell-de-at

### SSH config

