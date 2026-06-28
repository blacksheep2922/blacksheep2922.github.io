---
layout: post
title: Under the Inno Hood-How a Low-Detection Stealer Evades Analysis
categories:
  - C2C
---

## Overview

This analyzed malware is a sophisticated, multi-stage threat that initially deploys via a malicious Inno Setup installer configured to execute silently in the background while performing anti-analysis checks to evade virtual machines. Upon execution, it drops a C/C++ loader (`oJb1eg.exe`) and a suspiciously large 5MB icon file (`Devve.ico`)—likely containing encrypted payload data—into the local AppData directory. The loader profiles the system and injects the core payload, `rzamgo.k`, directly into memory as a DLL to bypass traditional disk-based antivirus scans. To further hinder reverse engineering, this core payload is heavily obfuscated using the commercial VMProtect packer. Once active, it spawns child processes to establish a command-and-control (C2) communication channel with the domain `tai8yan8jt.com` (192.168.139.2), which was offline at the time of analysis and exhibited a remarkably low 1/92 detection rate on VirusTotal.
