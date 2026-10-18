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

![[Pasted image 20261018193629.png)