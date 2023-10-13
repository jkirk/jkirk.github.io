---
title: "Lenovo ThinkPad X1 Carbon Gen 11: Debian/bookworm Post-Installation Journal"
create: 2023-10-07T08:37:03Z
date: 2023-10-07T08:37:03Z
draft: true
---

The Debian installation is easy. Configuring the system to my needs is much harder.

My goal is to bootstrap/deploy the most important programs and settings as quickly and as automatically as possible.
<!--more-->

## Linux Desktop Bootstrap

To make my Linux Desktop I need my dotfiles and some basic programs.

I developed [linux-desktop-bootstrap.sh](https://github.com/jkirk/linux-desktop-bootstrap) where the `git`, `etckeeper`, and `ansible-core` gets installed at first.
Then my dotfiles get deployed and the my base software selection gets installed.

```sh
❯ busybox wget -O - https://raw.githubusercontent.com/jkirk/linux-desktop-bootstrap/main/linux-desktop-bootstrap.sh | sh
```

See [jkirk/linux-desktop-bootstrap: Make your GNU/Linux Debian Desktop usable](https://github.com/jkirk/linux-desktop-bootstrap) for details.

## Migrate Data

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

Changed the hostname and `/etc/hosts`:

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

Changing the Volume Group name was a bit trickier:

When doing the following, the X-Server / Display Manager restarted:

```sh
❯ sudo vgrename vg0-predator vg0-tranquility
```

Adjusted `/etc/fstab` + `/boot/grub/grub.cfg` and updated initramfs:

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

## NetworkManager Profiles

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

## NetworkManager Mobile Broadband

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

### Cinnamon Keyboard Shortcuts

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

### Cinnamon Theme

![](screenshot_20231007T160540.png "Default Theme of GNOME Terminal")

I had the following settings in Debian bullseye

![](screenshot_20231007T115946.png "Cinnamon Themes on Debian/bullseye")

The same settings looked like this in Debian/bookworm:

![](screenshot_20231007T160332.png "Cinnamon Themes on Debian/bookworm")

I like a dark theme like Adapta-Nokoto, so after installing the theme I changed the Desktop + Application theme setting to `Adapta-Nokoto`.

![](screenshot_20231007T163736.png "GNOME Terminal in Adapta-Nokoto Theme")

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

### Thunderbird: Add-Ons

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

* End-To-End Encryption

  TODO

* Thunderbird Settings > Privacy & Security > Junk

  * When I mark messages as junk > Move them to the accounts "Junk" folder
  * Mark messages determeinded to be Junk as read
  * Enable adaptive junk filter loggin

* How to install Thunderbird Extensions / Add-ons automatically?

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

### SSH config
