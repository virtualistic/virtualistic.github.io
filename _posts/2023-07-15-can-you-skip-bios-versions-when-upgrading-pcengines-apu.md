---
title: "Can you skip versions when you upgrade a PC Engines APU2 bios?"
date: 2023-07-15T08:47:34+02:00
categories: [homelab,pfsense,pc engines]
tags: [pc engines, bios, pfsense, upgrade, apu2]
---

A quick note to people who are wondering if you can skip versions when upgrading the bios of (in my case) a PC Engines APU2

<!--more-->

## The Answer

YES YOU CAN!\
After googling a bit I could not find a definitive answer to this question.\
The APU2 was running version **4.12.0.4** and browsing the APU website, I noticed that the latest recommended version was **4.17.x**\
So after taking a few deep breaths, I decided to take the plunge, skip all versions between 4.13.x - 4.16.x and upgrade in one go to 4.17 using PFSense\
Recipe:
- Download the required version (4.17.0.3 in my case) from the [pcengines website](https://pcengines.github.io)
- In PFsense make sure you've installed the flashrom package using the command:
```bash
pkg install -y flashrom
```
- Upload the file through PFSense
- Diagnostics -> Command Prompt, Upload File
- to unpack the file:
```bash
tar -xvf /tmp/apu2_v4.17.0.3.rom.tar.gz
```
- Create a backup of your pfsense config just to be sure (Diagnostics -> Backup & Restore)
- Under "Execute Shell Command" type:
```bash
flashrom -w /tmp/apu2_v4.12.0.3.rom -p internal:boardmismatch=force
```
- After the bios upgrade, shutdown the box, remove the power and after waiting a few seconds, power it back on.\
Done!\
More background info on APU machines can be found on the excellent [TekLager website](https://teklager.se/en/knowledge-base/apu-bios-upgrade/).
