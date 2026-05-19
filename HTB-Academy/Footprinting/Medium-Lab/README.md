# Summary:
- Enumerate a server which is very common in every organization and attacker love to enumerate that service.
- Enumerate that server and find a password for user HTB.

## Scanning
- Scanning using nmap with all 65535 ports
`sudo nmap -sV -sC -Pn --disable-arp-ping --reason <target-ip> -oA FirstScan.out  -p- --stats=5s`

```Bash
# Nmap 7.99 scan initiated Sun May 17 07:57:50 2026 as: /usr/lib/nmap/nmap -sV -sC -Pn --disable-arp-ping --reason -oA FirstScan.out -p- --stats=5s <target-ip>
Nmap scan report for <target-ip>
Host is up, received user-set (0.16s latency).
Not shown: 65519 closed tcp ports (reset)
PORT      STATE SERVICE       REASON          VERSION
111/tcp   open  rpcbind       syn-ack ttl 127 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|_  100005  1,2,3       2049/udp6  mountd
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 127
2049/tcp  open  mountd        syn-ack ttl 127 1-3 (RPC #100005)
3389/tcp  open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WINMEDIUM
|   NetBIOS_Domain_Name: WINMEDIUM
|   NetBIOS_Computer_Name: WINMEDIUM
|   DNS_Domain_Name: WINMEDIUM
|   DNS_Computer_Name: WINMEDIUM
|   Product_Version: 10.0.17763
|_  System_Time: 2026-05-17T11:11:49+00:00
| ssl-cert: Subject: commonName=WINMEDIUM
| Not valid before: 2026-05-16T10:28:22
|_Not valid after:  2026-11-15T10:28:22
|_ssl-date: 2026-05-17T11:11:56+00:00; -1h00m01s from scanner time.
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
47001/tcp open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49679/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49680/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49681/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -59m58s, deviation: 2s, median: -59m58s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-05-17T11:11:51
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun May 17 08:12:30 2026 -- 1 IP address (1 host up) scanned in 880.24 seconds
```
`
Here we found out that mutliple ports are open and among of them also nfs(network file system) protocol is open, It is use for sharing network directory over the internet in linux or unix system in LAN network, Here everyone can access that directory and read or write as permission of them.

now here we are going to enumerate port 111,2049 that nfs service using nfs nmap scripts.
**command:**
`sudo nmap --script nfs* <target-ip> -p 111,2049 -sV -oA ScanningNFs.out`

**output**:
```Bash
┌──(kali㉿kali)-[~/Labs/Documentation/Footprinting/Medium_lab]
└─$ cat ScanningNFs.out.nmap 
# Nmap 7.99 scan initiated Sun May 17 08:33:48 2026 as: /usr/lib/nmap/nmap --script nfs* -p 111,2049 -sV -oA ScanningNFs.out <target-ip>
Nmap scan report for <target-ip>
Host is up (0.15s latency).

PORT     STATE SERVICE  VERSION
111/tcp  open  rpcbind  2-4 (RPC #100000)
| nfs-ls: Volume /TechSupport
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID         GID         SIZE   TIME                 FILENAME
| rwx------   4294967294  4294967294  65536  2021-11-11T00:09:49  .
| ??????????  ?           ?           ?      ?                    ..
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:28  ticket4238791283649.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:28  ticket4238791283650.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:28  ticket4238791283651.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:28  ticket4238791283652.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:28  ticket4238791283653.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:28  ticket4238791283654.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:29  ticket4238791283655.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:29  ticket4238791283656.txt
|_
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
| nfs-showmount: 
|_  /TechSupport 
| nfs-statfs: 
|   Filesystem    1K-blocks   Used        Available   Use%  Maxfilesize  Maxlink
|_  /TechSupport  41312252.0  16910440.0  24401812.0  41%   16.0T        1023
2049/tcp open  nlockmgr 1-4 (RPC #100021)
```
In this output we can see a directory named `TechSupport` listed in nfs.


Now Enumerating TechSupport Share using mount command

`sudo mount -t nfs <target-ip>/share-name /mnt/temp`

```Bash
┌──(root㉿kali)-[/mnt/temp]
└─# cat ticket4238791283782.txt                              
Conversation with InlaneFreight Ltd

Started on November 10, 2021 at 01:27 PM London time GMT (GMT+0200)
---
01:27 PM | Operator: Hello,. 
 
So what brings you here today?
01:27 PM | alex: hello
01:27 PM | Operator: Hey alex!
01:27 PM | Operator: What do you need help with?
01:36 PM | alex: I run into an issue with the web config file on the system for the smtp server. do you mind to take a look at the config?
01:38 PM | Operator: Of course
01:42 PM | alex: here it is:

 1smtp {
 2    host=smtp.web.dev.inlanefreight.htb
 3    #port=25
 4    ssl=true
 5    user="alex"
 6    password="<redacted>"
 7    from="<redacted>@web.dev.inlanefreight.htb"
 8}
 9
10securesocial {
11    
12    onLoginGoTo=/
13    onLogoutGoTo=/login
14    ssl=false
15    
16    userpass {      
17      withUserNameSupport=false
18      sendWelcomeEmail=true
19      enableGravatarSupport=true
20      signupSkipLogin=true
21      tokenDuration=60
22      tokenDeleteInterval=5
23      minimumPasswordLength=8
24      enableTokenJob=true
25      hasher=bcrypt
26      }
27
28     cookie {
29     #       name=id
30     #       path=/login
31     #       domain="10.129.2.59:9500"
32            httpOnly=true
33            makeTransient=false
34            absoluteTimeoutInMinutes=1440
35            idleTimeoutInMinutes=1440
36    }  
```

Using this information 
found an

> [!NOTE]
> user: <redacted>
> password: <redacted>


using this id pass connected with smb server with `rpcclient` Tool
`rpcclient -U "alex" <target-ip>`

``` Bash
┌──(kali㉿kali)-[~/Labs/Documentation/Footprinting/Medium_lab]
└─$ rpcclient -U "alex" <target-ip>
Password for [WORKGROUP\alex]:
rpcclient $> srvinfo
        <target-ip>  Wk Sv Sql NT SNT     
        platform_id     :       500
        os version      :       10.0
        server type     :       0x9007
rpcclient $> 
```

here we can see that its connected successfully. 

```Bash
┌──(kali㉿kali)-[~/Labs/Documentation/Footprinting/Medium_lab]                                                                                   
└─$ rpcclient -U "alex" <target-ip>                                                                                                            
Password for [WORKGROUP\alex]:                                                                                                                   
rpcclient $> srvinfo                                                                                                                             
        <target-ip>  Wk Sv Sql NT SNT                                                                                                          
        platform_id     :       500                                                                      
        os version      :       10.0                                                                     
        server type     :       0x9007                                                                   
rpcclient $> enumdomains                                                                                 
result was NT_STATUS_CONNECTION_DISCONNECTED                                                             
rpcclient $> querydomaininfo                                                                             
command not found: querydomaininfo                                                    
rpcclient $> qu                                                                       
queryaliasinfo        querydominfo          querysecret                               
queryaliasmem         querygroup            queryuser                                 
querydispinfo         querygroupmem         queryuseraliases                          
querydispinfo2        querymultiplevalues   queryusergroups                           
querydispinfo3        querymultiplevalues2  quit                                      
rpcclient $> query                                                                    
queryaliasinfo        querydominfo          querysecret                               
queryaliasmem         querygroup            queryuser                                 
querydispinfo         querygroupmem         queryuseraliases                          
querydispinfo2        querymultiplevalues   queryusergroups                           
querydispinfo3        querymultiplevalues2                                            
rpcclient $> queryd                                                                   
querydispinfo   querydispinfo2  querydispinfo3  querydominfo                          
rpcclient $> queryd                                                                   
querydispinfo   querydispinfo2  querydispinfo3  querydominfo                          
rpcclient $> querydominfo                                                             
result was NT_STATUS_CONNECTION_DISCONNECTED
rpcclient $> enumdomusers
result was NT_STATUS_CONNECTION_DISCONNECTED
rpcclient $> exit  
```
but as we can we dont have access of doing anything in smb server, but now we know this credentials are working then we can use this credentials in rdp or winrm.

now lets first enumerate rdp service running on port `3389` 
rdp is used to get gui access of windows, it is pre-installed in every windows machines.

command:
`xfreerdp /u:alex /p:"<redacted>" /v:<target-ip>`

/u: username
/p: password
/v: target ip


![[Pasted image 20260518131800.png]]

Hurray!!!, As we can see in the image using kali linux we get gui remote access of windows machine.
In the desktop a software is there named SQL Server
I tried to login in this file but not get success.
![[Pasted image 20260518135457.png]]

now we can find important things using this machine in its files and folders.


ok after successfully getting access of this windows server now i need to enumerate this windows machine to find something important files, after enumerating to many files and folder
inside this directory `/uers/alex/devshare` i found a file name `important` 
![[Pasted image 20260518135629.png]]

> sa:<redacted>
This is file is looking like storing a credential of username and password, How do i know it is username because in inside sql server we saw a name in login textbox `sa` ,
now i tried to login with this username and password to this sql server management but didnt able to get access, also i tried to accessing using different names like `admin,adminstrator,Admin,` but here also failed. 

I thought the password maybe in encrypted/hashed so i treid to find its algorithm but didn't able to get its algorithm.

Now after reading hint from hackthebox:
![[Pasted image 20260518140154.png]]
It says we have also an <redacted> account in windows system, so i know approached different method using those founds credentials i tried to login with rdp service.

```Bash
i tried firs this command with username sa.
┌──(kali㉿kali)-[~/Labs/Documentation/Footprinting/Medium_lab]
└─$ xfreerdp /u:sa /p:'<redacted>' /v:<target-ip>
[04:34:53:541] [6114:000017e2] [WARN][com.freerdp.client.common.cmdline] - [warn_credential_args]: Using /p is insecure
[04:34:53:541] [6114:000017e2] [WARN][com.freerdp.client.common.cmdline] - [warn_credential_args]: Passing credentials or secrets via command line might expose these in the process list
[04:34:53:541] [6114:000017e2] [WARN][com.freerdp.client.common.cmdline] - [warn_credential_args]: Consider using one of the following (more secure) alternatives:
[04:34:53:541] [6114:000017e2] [WARN][com.freerdp.client.common.cmdline] - [warn_credential_args]:   - /args-from: pipe in arguments from stdin, file or file descriptor
[04:34:53:541] [6114:000017e2] [WARN][com.freerdp.client.common.cmdline] - [warn_credential_args]:   - /from-stdin pass the credential via stdin
[04:34:53:541] [6114:000017e2] [WARN][com.freerdp.client.common.cmdline] - [warn_credential_args]:   - set environment variable FREERDP_ASKPASS to have a gui tool query for credentials
[04:34:53:548] [6114:000017e4] [WARN][com.freerdp.client.x11] - [load_map_from_xkbfile]:     : keycode: 0x08 -> no RDP scancode found
[04:34:53:548] [6114:000017e4] [WARN][com.freerdp.client.x11] - [load_map_from_xkbfile]: ZEHA: keycode: 0x5d -> no RDP scancode found
[04:34:54:193] [6114:000017e4] [WARN][com.freerdp.crypto] - [verify_cb]: Certificate verification failure 'self-signed certificate (18)' at stack position 0
[04:34:54:193] [6114:000017e4] [WARN][com.freerdp.crypto] - [verify_cb]: CN = WINMEDIUM
[04:34:54:193] [6114:000017e4] [ERROR][com.winpr.sspi.Kerberos] - [kerberos_AcquireCredentialsHandleA]: krb5_parse_name (Configuration file does not specify default realm [-1765328160])
[04:34:54:193] [6114:000017e4] [ERROR][com.winpr.sspi.Kerberos] - [kerberos_AcquireCredentialsHandleA]: krb5_parse_name (Configuration file does not specify default realm [-1765328160])
[04:34:54:502] [6114:000017e4] [ERROR][com.freerdp.core] - [nla_recv_pdu]: ERRCONNECT_LOGON_FAILURE [0x00020014]
[04:34:54:502] [6114:000017e4] [ERROR][com.freerdp.core.rdp] - [rdp_recv_callback_int][0x5610ff8f7060]: CONNECTION_STATE_NLA - nla_recv_pdu() fail
[04:34:54:502] [6114:000017e4] [ERROR][com.freerdp.core.rdp] - [rdp_recv_callback_int][0x5610ff8f7060]: CONNECTION_STATE_NLA status STATE_RUN_FAILED [-1]
[04:34:54:502] [6114:000017e4] [ERROR][com.freerdp.core.transport] - [transport_check_fds]: transport_check_fds: transport->ReceiveCallback() - STATE_RUN_FAILED [-1]
```
But it failed here

Now i tried username: <redacted>, and hurray!!! its logged in successfully. As an <redacted>.
![[Pasted image 20260518140555.png]]

Using SQL server management, after enumerating its directory i found password of HTB, Querying with sql query
![[Pasted image 20260518142328.png]]

password:<redacted>

Thats it of this LAB.
