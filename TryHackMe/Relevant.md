# Relevant
[Play](https://tryhackme.com/room/relevant)

## 1. Scanning & Enumeration
Starting off, we see
![An image of the Windows Web Server on port 80](https://imgur.com/UfRanXg.png)
An image of the Windows Web Server on port 80

### 1.1 Port Scanning
```
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV -A 10.10.252.107
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-18 22:32 EDT
Nmap scan report for 10.10.252.107
Host is up (0.30s latency).
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
|_  System_Time: 2021-05-19T03:14:38+00:00
| ssl-cert: Subject: commonName=Relevant
| Not valid before: 2021-05-18T03:03:51
|_Not valid after:  2021-11-17T03:03:51
|_ssl-date: 2021-05-19T03:15:17+00:00; +40m27s from scanner time.
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h04m26s, deviation: 3h07m50s, median: 40m26s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-05-18T20:14:38-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-05-19T03:14:40
|_  start_date: 2021-05-19T03:04:35

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 160.20 seconds
```

Quick information
![Information about port 139 and port 445 used for SMB protocol](https://i.imgur.com/Jd7M907.png)

[Source](https://www.varonis.com/blog/smb-port/)

I also did another full ports scan, just to be safe
```
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49663/tcp open  unknown
49667/tcp open  unknown
49669/tcp open  unknown
```

It is evident that we have some sneaky ports there too. Let's explore them 

### 1.2 Web enumeration
Gobuster returns an interesting page on `/*checkout*`, where we see

![A sceenshot of the runtime error on the checkout page](https://imgur.com/470t5p7.png)
A sceenshot of the runtime error on the checkout page

We can come back and look into this later. We did not see any more results.

### 1.3 SMB
Since SMB was open, let's explore that.
