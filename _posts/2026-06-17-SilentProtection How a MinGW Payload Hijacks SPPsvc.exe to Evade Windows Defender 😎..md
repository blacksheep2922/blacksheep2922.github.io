
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

![](<../../Pasted%20image%2020260617213839.png>)

![](<../../Pasted%20image%2020260617213858.png>)

and exports only 4 things that are callbacks. By looking at the import, I have some idea what the malware tries to do, checking for a debugger (IsDebuggerPresent). Also, some interesting things are there also, like sleep, suspend thread, and many more. 

![](<../../Pasted%20image%2020260617214031.png>)

