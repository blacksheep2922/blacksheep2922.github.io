---
categories:
  - AntiVM
  - Dropper
  - Privilege Escalation
  - Process Injection
tags:
  - Privilege_Escalation
  - Process_injection
title: Technical Analysis of Unpacked Dropper Token Privilege Escalation and Process Injection into Explorer.exe
layout: post
---
# Overview 
This report delivers a technical malware analysis of an uncompressed, 64-bit native Windows executable compiled with Microsoft Visual Studio 2019. Despite the complete absence of binary packing or code obfuscation, the sample implements a classic, multi-stage infection workflow focused on stealth execution, access token manipulation, registry-based persistence, and target process injection.

![](<Pasted image 20261018192255.png>)

MD5 : 3DE59C7196A3DA211BF49815D0C10336
SHA-256: 28fd91c15efccb7678249f03890952819453df461eef83ef1b12a562f9d35ac4

SHA1: 0f1e55c6975dde1e770033f59c6fd00445c0050a

File name : pee.exe

Size: 231936 (226 Kb)

![](<Pasted image 20261018193520.png>)


In this malware there is no protection that is used. That was strange according to today’s standards. The tool, as we can see in the screenshot, is Visual Studio 2019, and the language that is used is C. Also, when we open the malware in the CFX explorer, there are mainly 3 DLLs that are used

![](<Pasted image 20261018193629.png>)

By seeing only the DLL, we can assume what this malware will do, like the kernel32.dll and shell32. dll tells us almost the full story, but we have to see that this malware does

Let's see what we have seen in the **Import Directory**

### ADVAPI32.dll

![](<Pasted image 20261018193741.png>)

There are 6 imports in which some of them are red flags, or we can say they are used in malware.

* ***AdjustTokenPrivileges: It** is an API function used by malware to enable or disable specific privileges within a process’s access token, used in privilege escalation and defense evasion.

* OpenProcessToken: It is a function primarily used in malware to facilitate access token manipulation, a technique categorized under **MITRE ATT&CK T1134.**

* ***LookupPrivilegeValueEW**: It is a function that is used to retrieve the locally unique identifier (LUID) for a specific privilege name, which is a critical step in privilege escalation and access token manipulation.

These functions are used to do privilege escalation and other functions like registry interaction.

* **RegCloseKey**: It is mainly used to close a handle to a registry key.

* **RegCreateKeyW**: It is a function that is used to create a new registry key or open existing ones.

* **RegSetValueExW** is used to set the data and type of a specified value under a registry key. It is primarily abused for persistence, privilege escalation, and defense evasion.