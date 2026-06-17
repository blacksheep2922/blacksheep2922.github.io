
# Silent Protection: How a MinGW Payload Hijacks SPPsvc.exe to Evade Windows Defender 😎.

MD5 = 0F84697BF55CFC0A7BA358AEEF7BA8FD

SHA-1 = 364854D784BFD046C57675B16970A30C0C611D9E

SHA-256 =b998cca475ffaa2c1de38ed62f62dc0e79dccd4f8f66edead6abaf74525ca0e0

File Size = 135.50 KB (138752 bytes)

Original file name is ClientDispatcherAPI.exe

### Signature
![](<../assets/lib/SPPsvc/Pasted image 20260617212951.png>)

In this malware, the attacker uses a fake certification form from a company to show that it is legitimate software.

![](<../assets/lib/SPPsvc/Pasted image 20260617213645.png>)

- This tells about many things, like the malware using the MinGW compiler environment on Windows.
    
    - It is a GUI, which means it's not a console application; it will likely run in the background without popping up a command prompt window to alter the user.
        
    - The packer: compressed or packet data ".rdata," which means the IDA or Ghidra has high randomness in the .rdata section.

It only imports two DLLs from the Windows API, which are kernel32.dll (58 times) and msvrt.dll (38 times). Why? Because it uses different past in the kernel32.dll API to do its work.

![](<../assets/lib/SPPsvc/Pasted image 20260617213839.png>)

![](<../assets/lib/SPPsvc/Pasted image 20260617213858.png>)

and exports only 4 things that are callbacks. By looking at the import, I have some idea what the malware tries to do, checking for a debugger (IsDebuggerPresent). Also, some interesting things are there also, like sleep, suspend thread, and many more. 

![](<../assets/lib/SPPsvc/Pasted image 20260617214031.png>)

When I was looking in the strings of the malware (in Ghidra), I found some file location in which there is a mingw location. Why is it using the MinGW? This can be a clue that the malware author is using MinGW to compile rather than Visual Studio because MinGW statically links a lot of libraries. That's why there are many .C extension files in the strings.

**C:/crossdev/src/mingw-w64-v8-git/mingw-w64-libraries/winpthreads/src/rwlock.c**

![](<../assets/lib/SPPsvc/Pasted image 20260617214332.png>)


and one another thing is the rwlock jsut below it.

> “DEFINED 14001ce30 s_(((rwlock_t_*)*rwl)->valid_**_LI_14001ce30 ds "(((rwlock_t *)*rwl)->valid == LIFE_RWLOCK) && (((rwlock_t *)*rwl)->busy > 0)" "(((rwlock_t *)*rwl)->valid == LIFE_RWLOCK) && (((rwlock_t *)*rwl)->busy > 0)" string 77 true “**

What does that mean? The **relock is dereferencing a pointer to get to the read-write lock structure in memory.**

This whole line tells us that the malware isn’t a simple, single-threaded script. It manages multiple parallel threads of execution.

![](<../assets/lib/SPPsvc/Pasted image 20260617214535.png>)

This is the main code that is used to stop the malware from being detected and run after some time or, in this case, first turn off the defender of the victim's Windows.

![](<../assets/lib/SPPsvc/Pasted image 20260617214649.png>)

This is the whole flow of the malware: what changes, how it works. This is the overall flow of the malware.

The SPPsvc.exe is used for the hollowing process.

The malware is calling Windows APIs like QueryInformationThread, which is a low-level Windows native API that retrieves specific information about a thread, such as its page priority, entry point, or debugger status. The malware developer used this to hide a thread by hooking the API. It's an anti-debugger trick for the malware, and if a debugger detaches, it crashes the thread.

## LdrpInitialize

- It is an internal NTFLL function in Windows responsible for performing per-process initialization tasks when a new process is created. Malware often hooks this or uses it during process hollowing to hijack the execution of a suspended process, and this is used because the malware is suspended for a time, and when the defender gets deactivated, the malware does its work.
    

## NtAccessCheckAndAuditAlarm

- It is a native Windows system call (it is a part of NTAPI) that is used to validate whether a security description grants specific access rights to a client process.
    
- In this case this is used to change the registry of the victim's Windows to disable Windows Defender.