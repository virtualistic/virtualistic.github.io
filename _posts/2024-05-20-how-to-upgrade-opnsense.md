---
title: "How to upgrade OpnSense"
date: 2024-05-20T12:15:34+02:00
categories: [opnsense,upgrade,bectl,Boot Environment,IPTV]
tags: [OpnSense, upgrade, bectl, Boot Environment, IPTV]
---

A quick note on some best practices on how to upgrade OpnSense.

<!--more-->

## OpnSense Upgrade procedure

Yes, I moved to OpnSense. I needed a firewall with an up-to-date IGMP proxy package and PFSense's current implementation has an annoying [bug](https://forum.netgate.com/topic/184360/igmp-proxy-no-longer-works-reliably-after-2-7-1-update/80).  
Also the general direction that NetGate seems to be going has given me an extra motivation to bite the bullet and rebuild my firewall on OpnSense. 

*Recipe for  version upgrades*:  
As an example we are upgrading from 24.1.6 to 24.1.7. (For minor upgrades we could skip the creation of a new BE but we'll do it anyway)  

- Make sure you have a current config backup stored somewhere safe.  

- Check the current boot environments after logging in through ssh:  

```bash
root@opnsense:~ # bectl list
BE     Active Mountpoint Space Created
24.1.6 NR      /          1.23G 2024-04-20 19:07
```
The “N” stands for “Now” and the “R” is for “Reboot”, showing which Boot Environment is active now and after the next reboot.

- Create a new Boot Environment:   
```bash
root@opnsense:~ # bectl create 24.1.7
root@opnsense:~ # bectl activate 24.1.7
```
```bash
root@opnsense:~ # bectl list
BE     Active Mountpoint Space Created
24.1.6 N        -          1.23G 2024-04-20 19:07
24.1.7 R      /          0G 2024-05-20 10:00
```

- Reboot the firewall:  

```bash
shutdown -r now
```  

- Now perform the upgrade from the UI

- When we upgrade to a newer version, we can remove older Boot Environments like so:  

```bash
bectl destroy 24.1.6
```

- Done!
