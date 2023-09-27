---
title: "New Computer Checklist"
created: 2023-08-31T21:49:46+02:00
date: 2023-09-14T22:04:46+02:00
draft: false
toc: false
---

Before I set up a new computer, I usually follow this checklist.
<!--more-->

* Find out how to enter the BIOS settings before (usually Esc, F2, F12 or Enter)
* BIOS: Disable SecureBoot to be able to boot Grml (via Ventoy)
* BIOS: Enable VTx + VTd
* BIOS: Switch Ctrl + Fn key (on (Lenovo) Notebooks)
* BIOS: Take screenshot of System Information + BIOS Version
* BIOS: Set date and time (UTC if Debian only, local time if dual boot with Windows)
* BIOS (optional): Set Boot/BIOS password (after the Debian installation)
* Grml: Boot Grml (grml-full/sid daily), start grml-x and run `grml-hwinfo`, `inxi -xx -F` and save its output.[^1]
* Grml: Backup / create system image: `dd bs=1M count=1` (first 1 MB from MBR + PT), `ddrescue` + `ntfsclone -s` of other partitions
* If pre-installed with Windows:

  * Boot into the Windows Installer + retrieve Windows ProductKey (`wmic path softwarelicensingservice get OA3xOriginalProductKey`)
  * Shut down Windows Installer: `Shift-F10` (to open command prompt) > `shutdown /s /t 0` (to shut down)

* Install Firmware / BIOS update
* Partition the system disk (with or without Windows dual boot)
* Install Debian
* BIOS (optional): Re-enable SecureBoot

[^1]: `grml-hwinfo` v0.17.1, which includes the `inxi` output, has been included since grml-daily 2023-09-23 (build4288).
`xrandr` is always called, when `grml-hwinfo` is run with a X server environment.
