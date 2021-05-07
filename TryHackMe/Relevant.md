# Relevant
[Play](https://tryhackme.com/room/relevant)

## Recon
```
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV -A 10.10.6.215                                                                                                                                  130 ⨯
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-06 23:34 EDT
Nmap scan report for 10.10.6.215
Host is up (0.37s latency).
Not shown: 995 filtered ports
PORT     STATE SERVICE        VERSION
80/tcp   open  http           Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
135/tcp  open  msrpc          Microsoft Windows RPC
139/tcp  open  netbios-ssn    Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds   Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp open  ms-wbt-server?
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2021-05-07T03:36:50+00:00
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2021-05-06T03:23:42
|_Not valid after:  2021-11-05T03:23:42
|_ssl-date: 2021-05-07T03:37:29+00:00; +3s from scanner time.
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h24m02s, deviation: 3h07m50s, median: 2s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-05-06T20:36:49-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-05-07T03:36:51
|_  start_date: 2021-05-07T03:24:15

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 172.66 seconds
```

gobuster did not show anything interesting.

Looking into a popular exploit, Eternal Blue, this machine does not look like its vulnerable. Dead end!


Enumeration time again. This time, for all the ports. Keep that running in the background. Meanwhile, let's have a look at smb.

```
┌──(kali㉿kali)-[/tmp]
└─$ smbclient -L 10.10.6.215     
Enter WORKGROUP\kali's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        nt4wrksv        Disk      
SMB1 disabled -- no workgroup available
```

Very nice. How about, 

```
┌──(kali㉿kali)-[/tmp]
└─$ smbclient //10.10.6.215/    
Enter WORKGROUP\kali's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Jul 25 17:46:04 2020
  ..                                  D        0  Sat Jul 25 17:46:04 2020
  passwords.txt                       A       98  Sat Jul 25 11:15:33 2020

                7735807 blocks of size 4096. 4937977 blocks available
smb: \> get passwords.txt 
getting file \passwords.txt of size 98 as passwords.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
smb: \> 
                                                                                  
┌──(kali㉿kali)-[/tmp]
└─$ cat passwords.txt        
[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk
```

Okay. Let's shove them into a base64 decoder.

```
┌──(kali㉿kali)-[/tmp]
└─$ echo Qm9iIC0gIVBAJCRXMHJEITEyMw== | base64 -d
Bob -!P@$$W0rD!123                                                                                  
┌──(kali㉿kali)-[/tmp]
└─$ echo QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk | base64 -d
Bill - Juw4nnaM4n420696969!$$$ 
```

Sweet. Now, let's try logging in. ... How? idk man wtf.


```
┌──(kali㉿kali)-[~]
└─$ nmap -p- -T4 10.10.6.215 -v
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-06 23:47 EDT
Initiating Ping Scan at 23:47
Scanning 10.10.6.215 [2 ports]
Completed Ping Scan at 23:47, 0.26s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 23:47
Completed Parallel DNS resolution of 1 host. at 23:47, 0.00s elapsed
Initiating Connect Scan at 23:47
Scanning 10.10.6.215 [65535 ports]
Discovered open port 3389/tcp on 10.10.6.215
Discovered open port 80/tcp on 10.10.6.215
Discovered open port 139/tcp on 10.10.6.215
Discovered open port 445/tcp on 10.10.6.215
Discovered open port 135/tcp on 10.10.6.215
Connect Scan Timing: About 0.39% done
Connect Scan Timing: About 4.81% done; ETC: 00:08 (0:20:26 remaining)
Connect Scan Timing: About 11.18% done; ETC: 00:01 (0:12:11 remaining)
Connect Scan Timing: About 19.61% done; ETC: 23:57 (0:08:20 remaining)
Discovered open port 49663/tcp on 10.10.6.215
Connect Scan Timing: About 29.34% done; ETC: 23:55 (0:06:06 remaining)
Connect Scan Timing: About 37.49% done; ETC: 23:55 (0:05:03 remaining)
Connect Scan Timing: About 51.30% done; ETC: 23:54 (0:03:21 remaining)
Discovered open port 49669/tcp on 10.10.6.215
Connect Scan Timing: About 62.71% done; ETC: 23:53 (0:02:24 remaining)
Discovered open port 49667/tcp on 10.10.6.215
Connect Scan Timing: About 74.04% done; ETC: 23:53 (0:01:35 remaining)
Connect Scan Timing: About 81.13% done; ETC: 23:54 (0:01:16 remaining)
Completed Connect Scan at 23:53, 373.28s elapsed (65535 total ports)
Nmap scan report for 10.10.6.215
Host is up (0.25s latency).
Not shown: 65527 filtered ports
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49663/tcp open  unknown
49667/tcp open  unknown
49669/tcp open  unknown

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 373.60 seconds
```

Ah so we do have things at the bottom hmmm ... Let's run scans specific for the ports.

```
┌──(kali㉿kali)-[~]
└─$ nmap -p 49663 -sC -sV 10.10.6.215                                       130 ⨯
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-07 00:03 EDT
Nmap scan report for 10.10.6.215
Host is up (0.67s latency).

PORT      STATE SERVICE VERSION
49663/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-IIS/10.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 64.81 seconds
```

```
┌──(kali㉿kali)-[~]
└─$ nmap -p 49667 -sC -sV 10.10.6.215
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-07 00:07 EDT
Nmap scan report for 10.10.6.215
Host is up (0.45s latency).

PORT      STATE SERVICE VERSION
49667/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 62.53 seconds
```

```
┌──(kali㉿kali)-[~]
└─$ nmap -p 49669 -sC -sV 10.10.6.215
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-07 00:12 EDT
Nmap scan report for 10.10.6.215
Host is up (0.28s latency).

PORT      STATE SERVICE VERSION
49669/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 61.78 seconds
```

Ok so we will take RPC ports lite, but yo, what's up with 49663! We have httpd :D Let's go and see.

![](https://i.imgur.com/0jH6vro.png)


![](https://i.imgur.com/kp4IpqD.png)


Okay, so both are related. Now, how to get into the system? If we could somehow upload a reverse shell - then, we could just go to the file in the browser, to start it. ez.
