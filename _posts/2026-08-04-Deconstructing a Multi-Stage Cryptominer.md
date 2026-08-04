---
layout: post
title: "Deconstructing a Multi-Stage Cryptominer: WaaSMedicSvc Evasion and Mining Pool Analysis"
categories:
  - C2C
  - AntiVM
  - Self-delete
  - VMProtect
  - Cryptominer
  - Cryptojacking
tags:
  - Crytominer
  - C2C
---
## **Overview**

This sample is a heavily packed, anti-VM/anti-debug cryptomining trojan that evaded most AV engines (2/91 on VirusTotal). Static analysis was hampered by obfuscation, so the investigation relied mainly on dynamic analysis: the binary self-deletes only once internet connectivity is confirmed, and it drops four automated PowerShell scripts that first attempt (and fail, then retry) to disable Windows Defender via exclusion rules, and successfully hijack registry permissions to disable the WaaSMedicSvc service so Windows Update can't self-repair, leaving the system unpatched and persistently vulnerable. Network analysis revealed the malware communicating with a C2 server at 141.94.96.144 (a French datacenter host, [ns31430818.ip-141-94-96.eu](http://ns31430818.ip-141-94-96.eu/)) using Stratum-style mining pool protocol strings, confirming its purpose as a cryptomining payload that prioritizes defense evasion and persistence over stealth in its (somewhat crude, blindly-retrying) automation logic.

![](<../assets/lib/Miner/Pasted%20image%2020260804210059.png>)

**SHA256 =a94e9aca1aca0c7e006a0d8684c5423b1a3bd7e48734eee4f12f0caa3b5d901a**

**MD5 = be2f08950440b3bb987fc5d0999b3f1f**

**SHA1=689a96b72e20cc501fa145637d1cdc3f76d68a3c**

**FILE SIZE = 5,210,112**

**Original file name =LockAppHost14a02b.exe**
![](../../Pasted%20image%2020260804210200.png>)