This CTF involves domains and likely subdomains, so I went ahead and added `futurevera.thm` to my `/etc/hosts` file.
Enumeration

# Enumeration
The description says:
> " This challenge revolves around subdomain enumeration."
>  so I'll try to focus on that.
## Nmap
First, let's run an Nmap scan to get an idea of what we're dealing with:
```bash
$ nmap futurevera.thm
..Snip..
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

```

Let’s try a more evasive scan:
```bash
$ nmap -p22,80,443 -A futurevera.thm
..Snip..
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 dd:29:a7:0c:05:69:1f:f6:26:0a:d9:28:cd:40:f0:20 (RSA)
|   256 cb:2e:a8:6d:03:66:e9:70:eb:96:e1:f5:ba:25:cb:4e (ECDSA)
|_  256 50:d3:4b:a8:a2:4d:1d:79:e1:7d:ac:bb:ff:0b:24:13 (ED25519)
80/tcp  open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Did not follow redirect to https://futurevera.thm/
|_http-server-header: Apache/2.4.41 (Ubuntu)
443/tcp open  ssl/http Apache httpd 2.4.41
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: FutureVera
| ssl-cert: Subject: commonName=futurevera.thm/organizationName=Futurevera/stateOrProvinceName=Oregon/countryName=US
| Not valid before: 2022-03-13T10:05:19
|_Not valid after:  2023-03-13T10:05:19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

## Subdomain Enumeration
DNS port 53 is closed, so we’ll need to brute-force subdomains:
```bash
$ ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -mc all -H "Host: FUZZ.futurevera.thm" -u https://futurevera.thm/ -fs 4605
```

**Results:**
```
support                 [Status: 421, Size: 411, Words: 38, Lines: 13, Duration: 53ms]
blog                    [Status: 421, Size: 408, Words: 38, Lines: 13, Duration: 48ms]
```

Time to add these subdomains to /etc/hosts and check them out.
## Directory Bruteforce
I didn’t find much of interest on any of the subdomains, so I tried a directory scan:
```bash
$ dirb https://support.futurevera.thm/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Wed May 14 19:18:08 2025
URL_BASE: https://support.futurevera.thm/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: https://support.futurevera.thm/ ----
==> DIRECTORY: https://support.futurevera.thm/assets/                                                  
==> DIRECTORY: https://support.futurevera.thm/css/                                                     
+ https://support.futurevera.thm/index.html (CODE:200|SIZE:1522)                                       
==> DIRECTORY: https://support.futurevera.thm/js/                                                      
+ https://support.futurevera.thm/server-status (CODE:403|SIZE:288)
```

The other subdomains (`www` and `blog`) returned identical results. The directories had listing enabled but didn’t contain anything useful.
## SSL Certificate
Neither `blog.futurevera.thm` nor `futurevera.thm` revealed anything interesting, but `support.futurevera.thm` did:

![[Pasted image 20250515131040.png]]

Visiting the site redirected me to:

http://flag{beea0d6edfcee06a59b83fb50ae81b2f}.s3-website-us-west-3.amazonaws.com/

# Conclusion
This one felt like grasping at straws. I brute-forced for nearly two hours before realizing the clue was in the SSL certificate the whole time.

**What did I learn?**
-    Check SSL certs immediately when a site has one.
-    brute-forcing isn’t the best approach to web enumeration.