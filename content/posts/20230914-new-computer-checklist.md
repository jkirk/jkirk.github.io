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
* BIOS: Set date and time (UTC, local time if dualboot with Windows)
* BIOS: Set Boot/BIOS password (after finishing the installation)
* Grml: Boot Grml (daily) and save `grml-hwinfo`, `xrandr` (via grml-x) + `inxi -xx -F` output
* Grml: Backup / create system image: `dd bs=1M count=1` (first 1 MB from MBR + PT), `ddrescue` + `ntfsclone -s` of other partitions
* Boot into Windows Installer + retrieve Windows ProductKey (`wmic path softwarelicensingservice get OA3xOriginalProductKey`)
* Shut down Windows Installer: `Shift-F10` (to open command prompt) > `shutdown /s /t 0` (to shut down)
* Install Firmware / BIOS update
* Install Debian
