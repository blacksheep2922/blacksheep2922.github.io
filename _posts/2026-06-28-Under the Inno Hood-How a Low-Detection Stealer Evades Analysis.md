---
layout: post
title: Under the Inno Hood-How a Low-Detection Stealer Evades Analysis
categories:
  - C2C
  - VMProtect
  - Dropper
  - AntiVM
---

## Overview

This analyzed malware is a sophisticated, multi-stage threat that initially deploys via a malicious Inno Setup installer configured to execute silently in the background while performing anti-analysis checks to evade virtual machines. Upon execution, it drops a C/C++ loader (`oJb1eg.exe`) and a suspiciously large 5MB icon file (`Devve.ico`)—likely containing encrypted payload data—into the local AppData directory. The loader profiles the system and injects the core payload, `rzamgo.k`, directly into memory as a DLL to bypass traditional disk-based antivirus scans. To further hinder reverse engineering, this core payload is heavily obfuscated using the commercial VMProtect packer. Once active, it spawns child processes to establish a command-and-control (C2) communication channel with the domain `tai8yan8jt.com` (192.168.139.2), which was offline at the time of analysis and exhibited a remarkably low 1/92 detection rate on VirusTotal.


![](<../assets/lib/ojb1leg/flowchart.png>)

Original File name = 8Sf0.exe

Some other names = 2026-6-26 1.exe , 7317d297686d154b4d78217e100df5f57949f05efe095f1a017b5988cddef98b.exe

MD5=C21C6962C9902DDBF4D08537EA7D96A4

SHA-1=2C707CA426222F790DC10216F9784127B386BF75

SHA-256 = 7317d297686d154b4d78217e100df5f57949f05efe095f1a017b5988cddef98b

File size = 23.14 MB (24260952 bytes)

File Type = Portable Executable 32

In this sample, we didn’t find any signatures in either Detect It Easy or in PE Explorer.

![](<../assets/lib/ojb1leg/Pasted image 20260628232644.png>)

But what we found in Detect It Easy was very interesting.

![](<../../Pasted%20image%2020260628232813.png>)

In this picture, we can see that the executable is packed and obfuscated, which means if we try to add this to Ghidra or IDA, we will not have that many things that we want, and the installer is **an Inno Setup module.**

