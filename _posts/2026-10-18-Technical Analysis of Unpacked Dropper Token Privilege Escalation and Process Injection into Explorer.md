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

* **OpenProcessToken:** It is a function primarily used in malware to facilitate access token manipulation, a technique categorized under **MITRE ATT&CK T1134.**

* ***LookupPrivilegeValueEW**: It is a function that is used to retrieve the locally unique identifier (LUID) for a specific privilege name, which is a critical step in privilege escalation and access token manipulation.

These functions are used to do privilege escalation and other functions like registry interaction.

* **RegCloseKey**: It is mainly used to close a handle to a registry key.

* **RegCreateKeyW**: It is a function that is used to create a new registry key or open existing ones.

* **RegSetValueExW** is used to set the data and type of a specified value under a registry key. It is primarily abused for persistence, privilege escalation, and defense evasion.
### KERNEL32.dll

In kernel32.dll there are not so special functions used; some simple functions are used that are used by simple programs. WriteProcessMemory and WriteConsoleW are some of the functions that can be used to do some nasty things.

### SHELL32.dll

SHGetFolderPathW: It is used to dynamically locate special system directories; it allows software to adapt to different user profiles and Windows versions without relying on or brittle, hardcoded paths, thereby increasing its portability and stealth.

Now when we open the executable in a debugger like Ghidra, we see each one of them working in the code.

![](<Pasted image 20261018193906.png>)

as we see in the import section the ReCloseKey is used to chagne the registry
![](<Pasted image 20261018193939.png>)

when we looks down we found what the whole malware is doing

![](<Pasted image 20261018194011.png>)
The file path it opens is HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run.

is represented by 0xffffffff80000001. It then writes the path of the newly copied executable to a registry key named "Syshost." This ensures the malware runs automatically every time the user logs in.

![](<Pasted image 20261018194043.png>)

**Execution of Payload:** It prepares to launch the copied file using CreateProcessW. It sets wShowWindow to 0 (SW_HIDE) and uses the creation flag 0x8000000 (CREATE_NO_WINDOW). This executes the new malware instance completely invisibly in the background.

Now when we look at the **LookupPrivilegeValueW**

We found what we have seen in the PE Explorer, and we predicted

This code is used to process injection

![](<Pasted image 20261018194119.png>)

In this we can see that this code is used for process injection, especially targeting "explorer.exe." This technique is used by malware developers to hide malicious activity within a legitimate, twisted system process.

second

![](<Pasted image 20261018194145.png>)
In this, the main goal of the malware is to check what privileges it has, and then it creates the process, as we see in the above, that it is doing the process injection. In this code, they are creating the role to do that.

Now we get the whole idea, but we also have the SHELL32. dll lets us analyze it too

![](<Pasted image 20261018194213.png>)

- **Directory Resolution (SHGetFolderPathW):** The code calls SHGetFolderPathW with the second parameter set to 0x1a. In the Windows API, 0x1a is the hexadecimal value for the constant CSIDL_APPDATA. This instructs the system to return the path to the current user's Roaming AppData folder (e.g., C:\Users\<username>\AppData\Roaming).
- **String Construction:** The do...while loop calculates the string length of the returned directory path. It then uses internal C++ standard library functions (FID_conflict:append) to append a backslash (likely DAT_14002db1c) and the hardcoded string L"syshost.exe."
- **Memory Management:** The verbose blocks checking if (7 < uStack_240) and calling thunk_FUN_140005458 are the decompiler's representation of memory cleanup for C++ std::wstring objects. It is checking if the string was long enough to require heap allocation, and if so, freeing that memory before the function returns.
- **Return:** The final constructed path (e.g., C:\Users\<username>\AppData\Roaming\syshost.exe) is returned to the caller via param_1, which acts as an out-parameter pointer for the constructed object.

