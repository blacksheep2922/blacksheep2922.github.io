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

**Original File name = 8Sf0.exe**

Some other names = **2026-6-26 1.exe , 7317d297686d154b4d78217e100df5f57949f05efe095f1a017b5988cddef98b.exe**

**MD5=C21C6962C9902DDBF4D08537EA7D96A4**

**SHA-1=2C707CA426222F790DC10216F9784127B386BF75**

**SHA-256 = 7317d297686d154b4d78217e100df5f57949f05efe095f1a017b5988cddef98b**

File size = **23.14 MB (24260952 bytes)**

File Type = **Portable Executable 32**

In this sample, we didn’t find any signatures in either Detect It Easy or in PE Explorer. :)

![](<../assets/lib/ojb1leg/Pasted image 20260628232644.png>)

But what we found in Detect It Easy was very interesting.

![](<../assets/lib/ojb1leg/Pasted image 20260628232813.png>)

In this picture, we can see that the executable is packed and obfuscated, which means if we try to add this to Ghidra or IDA, we will not have that many things that we want, and the installer is **an Inno Setup module.**

So now I have used another tool for the installer. InnoExtractor is a GUI tool in which we can extract things without running the main executable. This helps me to see what's inside the executable.

![](<../assets/lib/ojb1leg/Pasted image 20260628233736.png>)

And I got some interesting files, and some of these files are very interesting, like **devve.ico** and **rzamGo.K,** also the install_script.iss and CodeSection.txt, and when we open the install_script in any text editor, it tells us many things about the installation process.

![](<../assets/lib/ojb1leg/Pasted image 20260629094156.png>)

![](<../assets/lib/ojb1leg/Pasted image 20260629094238.png>)
From these images I can find many details, like what the drop location is, the level of privilege required, and the evasion tactic.

### The drop location

**DefaultDirName: {localappdata}\Local\assembly\d13\GDQ8\O30x**

Instead of installing to Program Files like normal software, it is hiding deep within the user's AppData\Local directory. By creating fake folders named "assembly," it is trying to blend in with legitimate Windows .NET framework directories so the user won't notice.

### Evasion Tactics

Uninstallable =no  : This is a classic malware behavior . it intentionally breaks the ability for the user to remove it via the standard "Add/Remove Program" menu in Windows.

Privileges Required = lowest : The malware developer configured this to run with standard user privileges. This means the installer won't rigger a noisy User Account Control(UAC) prompt asking for admin rights. 
![](<../assets/lib/ojb1leg/Screenshot%202026-06-27%20182039.png>)

The important part in this picture is "Run" -> it tells about where the real malware will install itself as well as some more things to see, like the flag sections.

- **Flags: hidewizard nowait***
    
    - The hidewizard: this flag hides the installer, not to wait for the launcher program to terminate before proceeding to the next step or hiding the installation.
        
    - nowait: This flag allows the installer to launch an application silently in the background.


## The main Hero -The  oJb1eg.exe

MD5 = **11a5de0ad53c4e1e9b5d5404a9c9e755**

SHA-1 = **6b5368715772a9368960beff02c575144481f277**

SHA-256 =**2e42798c8bfb951de668a4df82b7fb81eb9fd98ae5b0000d75a3422665c05f85
**
Imports : **rzamGo.k** 

File Size : **1.16MB**

VirusTotal Score :) dammn.

![](<../assets/lib/ojb1leg/Pasted image 20260629100833.png>)

![](<../assets/lib/ojb1leg/Screenshot 2026-06-27 182741.png>)This tells about many things, like we can't see DLLs and can't put them in the debugger like Ghidra and IDA; it is developed in C++ and Visual Studio.

and it is used protector : **VMProtect**
* The VMProtect is an incredibly aggressive commercial-grade packer. It doesn't just encrypt the file; it turns real assembly instructions into their own visual bytecode that only the internal engine understands. 

![](<../assets/lib/ojb1leg/Pasted image 20260629101758.png>)

I couldn't find much of it information because the code was obfuscated and packed to let me try something new.

new when I run the malware in the 64x dbg and in the symbol table I found this: obj1eg.exe and rzamgo.k  
That confirms that the obj1eg is just a simple stub or runner; its entire job is to lead rzamGo.k into memory. The windows are treating it like a dynamic link library.