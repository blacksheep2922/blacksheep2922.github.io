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

**Original file name =LockAppHost14a02b.exe

![](<../assets/lib/Miner/Pasted image 20260804210200.png>)

By looking at this image, we can see that doing static analysis would be difficult because debuggers like Ghidra and IDA will not give the output we want. So we have to change our approach; we have to shift to the x64dbg because it has the ScyllaHide plugin that helps us in this case.

![](<../assets/lib/Miner/Pasted%20image%2020260804210245.png>)

While looking at the CFF explorer, we found out that the malware only imports two DLLs: **KERNEL32.dll** and **USER32.dll.**

Because of the projection, the generic because of that, the structural section of the PE file lies in .txt .data and has unusual names or permissions.

So I changed the approach. I put the malware into the debugger x64dbg and turned on the **ScyllaHide** plugin that gives me a lot of information about it.

![](<../assets/lib/Miner/Pasted%20image%2020260804210318.png>)

While looking in the x64dbg, I found some things entering the malware using the function that are not well documented by Microsoft (there was a meme going in my mind: fukkkk, Microsoft), but there are some more interesting things in the debugger.

The MaxLoaderThreads is a Windows registry setting used to control the parallel loading of DLLs during process initialization located under **HKEY_LOCAL_MACHINE\SOFTWARE\. Microsoft\Windows NT\CurrentVersion\Image File Execution Options**

Setting to 1 disables parallel loading, forcing the process to load DLLs sequentially using only the main thread, and I also find some interesting terms like "RaiseExceptionOnPossibleDeadlock" and "tracingFlags UseImpersonatedDeviceMap."

and MaxDeadActivationContext

![](<../assets/lib/Miner/Pasted%20image%2020260804210341.png>)

**L"\\Registry\\Machine\\Software\\Microsoft\\Windows NT\\CurrentVersion\\AppCompatFlags\\Compatibility Assistant\\**

This is a PCA—it belongs to the Windows Program Compatibility Assistant.

This is a built-in Windows feature designed to monitor programs as they run. If a program crashes, fails to install, or has known compatibility issues with the current version of Windows, the PCA intervenes.

It also detects the debug environment, and it is self-deleted.

Malware is an anti-VM and debug; because of that, we can’t see that much of it, so we have to do dynamic analysis.

## Dynamic Analysis ( Level 2 - The Playground )

so when the malware run in the virtual machine, it create two child process sc.exe and sonhost.exe which both are legitimate windows process , so no malware is downloaded but the main things s that the malware delete itself after running it, because of the detection it is checking that it is running on the virtual machine or not but we got the information we want.

![](<../assets/lib/Miner/Pasted image 20260804210508.png>)

and this is the graph of the malware

![](<../assets/lib/Miner/Pasted image 20260804210535.png>)

These are control flow diagrams of the malware. We can’t see that much because of the obfuscation that is created by the malware developer, but we have also found that the malware created some powerful fields that have their own commands in them. There are a total of 4 files that are created.

**PowerShell_transcript.DESKTOP-4EUNG44.5zLDQykN.20260803083750.txt**

**PowerShell_transcript.DESKTOP-4EUNG44.5zLDQykN.20260803083752.txt**

**PowerShell_transcript.DESKTOP-4EUNG44.5zLDQykN.20260803084011.txt**

**PowerShell_transcript.DESKTOP-4EUNG44.5zLDQykN.20260803083909.txt**

These are the four files that are created, and each text file has a different role in it, as we can see some glimpses of it in the debugger in that.

