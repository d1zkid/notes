## Basic use
`nmap [target IP]`
 
 - `-sV`: _show Version_
 - `-sC`: _safe Scripts_
 - `-p22,80`: only scans port 22 and 80(`-p-`: _scan all ports_)
 - `-oN filename`: output the .nmap file
 - `-O`: OS detection _requires sudo priveleges_
 - `-A`: Aggressive scan _may get detected_

## Advanced options
*most require **sudo priveleges***
 - `--script [script//script category]`: uses a _script_ or a _script category_
 - `-D RND:5`: uses 5 random decoy IP's
 - `-n`: disables DNS resolution
 - `-g53`: uses a different source port _useful for DNS servers_
 - `-e tun0`: uses specified network interface

## Script category's
can be used with `--script [category]`  

| Category    | Description                                                                                                                             |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `auth`      | Determination of authentication credentials.                                                                                            |
| `broadcast` | Scripts, which are used for host discovery by broadcasting and the discovered hosts, can be automatically added to the remaining scans. |
| `brute`     | Executes scripts that try to log in to the respective service by brute-forcing with credentials.                                        |
| `default`   | Default scripts executed by using the `-sC` option.                                                                                     |
| `discovery` | Evaluation of accessible services.                                                                                                      |
| `dos`       | These scripts are used to check services for denial of service vulnerabilities and are used less as it harms the services.              |
| `exploit`   | This category of scripts tries to exploit known vulnerabilities for the scanned port.                                                   |
| `external`  | Scripts that use external services for further processing.                                                                              |
| `fuzzer`    | This uses scripts to identify vulnerabilities and unexpected packet handling by sending different fields, which can take much time.     |
| `intrusive` | Intrusive scripts that could negatively affect the target system.                                                                       |
| `malware`   | Checks if some malware infects the target system.                                                                                       |
| `safe`      | Defensive scripts that do not perform intrusive and destructive access.                                                                 |
| `version`   | Extension for service detection.                                                                                                        |
| `vuln`      | Identification of specific vulnerabilities.                                                                                             |
