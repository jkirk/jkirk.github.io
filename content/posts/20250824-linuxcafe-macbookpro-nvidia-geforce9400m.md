---
title: Legacy Nvidia-Treiber unter Linux
date: 2025-08-24T13:04:40Z
toc: false
---

Wie bekommt man den Legacy Treiber für die Nvidia Geforce 9400M unter Linux wieder zum Laufen?
<!--more-->

## Das Problem

Ein Linux-Cafe-User hat uns folgendes Problem geschickt:

> MacBook Pro: Wie bekomme ich den Nvidia-Treiber zum Laufen? (Früherer Linux-Kernel von 2020?) —> MacBook Pro "Core 2 Duo" 2.26 13" (SD/FW) Specs (Mid-2009 13", MB990LL/A, MacBookPro5,5, A1278, 2326\*), NVIDIA GeForce 9400M, Ubuntu 20.04 LTS hat zuletzt NVIDIA 340 unterstützt, aber jetzt wird 340 nicht mehr unterstützt. Aktuell geht der Display Out port nicht mehr und video abspielen ist fast unmöglich. Beides hat zuvor funktioniert.
>
> Gibt es andere Distributionen, die die Nvidia-Karte unterstützen könnten?

## Recherche

Ich habe etwas recherchiert und folgenden Debian-Bug-Eintrag und Link gefunden:

* [#973599: nvidia-graphics-drivers-legacy-340xx: EoL driver should not be released with bullseye](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=973599)
* [NVIDIA Legacy 340xx - Debian User Forums](https://forums.debian.net/viewtopic.php?t=149950)

Weil Nvidia den Support für Legacy GPUs [eingestellt](https://nvidia.custhelp.com/app/answers/detail/a_id/3142) hat, hat der Debian-Maintainer beschlossen, den Treiber nicht für Debian/stable (damals Debian/bullseye) zuzulassen.

Trotzdem hat er zugesagt, den Treiber in Debian/sid zu belassen und auf neue Kernel zu aktualisieren (solange der Aufwand für ihn vertretbar ist).

Das bedeutete auch, dass der offizielle Upgrade-Pfad wohl darin bestand, vor dem Debian-Upgrade (von Debian/buster auf Debian/bullseye) auf den freien Nouveau-Treiber umzusteigen.

Die Nvidia Geforce 9400M Grafikkarte wird zwar vom Treiber [unterstützt](https://nouveau.freedesktop.org/CodeNames.html#NV50), glücklich scheinen die Leute damit aber nicht zu sein.[^1]

[^1]: https://forums.debian.net/viewtopic.php?p=745277#p745277

Die Situation ist semi-optimal.
Ich habe Guides gefunden, wie man den Legacy-Treiber unter Debian/stable installieren kann:

* [How to install the Nvidia legacy driver (340xx or 390xx) on Debian 13 Trixie](https://gist.github.com/Anakiev2/8d62e261c66554d3012bc7ff855a22a7)
* [How to install nvidia-legacy-340xx-driver on Debian 12 Bookworm](https://gist.github.com/Anakiev2/b828ed2972c04359d52a44e9e5cf2c63)
* [Install Nvidia legacy driver 340xx on Debian 11 Bullseye](https://gist.github.com/oprizal/998635a2ff5cbecb0519455c12b2994f)

Der Guide für Debian/trixie wurde erst vor wenigen Stunden online gestellt,
trotzdem müsste man es wohl zuerst einmal ausprobieren.

Alternativ könnte man es auch direkt mit Debian/sid probieren.

## Andere Distributionen

Ubuntu hat den Legacy-Treiber beim Upgrade von 20.04LTS auf 22.04LTS automatisch auf den Nouveau-Treiber [umgestellt](https://packages.ubuntu.com/jammy/nvidia-340).

Laut folgendem Blog gibt es für Ubuntu und MX Linux auch die Möglichkeit den Legacy Nvidia-340er-Treiber zu installieren:

* [These Linux Distributions still Support the Nvidia 340 Driver and How to Install it - IT & Internet - MidnightMaster95.com](https://midnightmaster95.com/main/it-internet/these-linux-distributions-still-support-the-nvidia-340-driver-and-how-to-install-it-r36/)

*MX Linux basiert auf Debian. Möglicherweise kann man das [MX Linux Repository](https://mxrepo.com/mx/repo/pool/non-free/n/nvidia-graphics-drivers-legacy-340xx/) einbinden und den Legacy Treiber von dort installieren.*
