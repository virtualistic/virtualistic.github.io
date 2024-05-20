---
title: "Best tool to migrate Windows settings to a different PC"
date: 2024-05-20T10:15:34+02:00
categories: [windows 10,windows 11,migration,user profile]
tags: [windows 10, windows 11, user profile, migration]
---

A reminder for myself what tool I used to migrate my windows settings from my old pc to a newer one.

<!--more-->

## The Tool

The tool is called Transwiz.  
Transwiz brings over user profiles, no support for Azure accounts nor data or applications which is fine for me.
Recipe:
- Download the latest version  [from here](https://www.forensit.com/move-computer.html)
- On your old and new computer, create a temporary local admin account and login to that account on your old computer.  
- There is no install, just run the Transwiz program and follow the wizard.  
- Save the profile of your choosing to a zip file and transfer it to your new computer.  
- Start Transwiz on your new computer when logged in as the temporary admin account and import the zip file  
- After importing the profile, reboot the new computer.  
- Done!
