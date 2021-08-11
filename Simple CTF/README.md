Simple CTF
====================

> Armand Alvarez | 11 August 2021

> [github.com/Armand-Alvarez](github.com/Armand-Alvarez)

> [Room link on TryHackMe](https://tryhackme.com/room/easyctf)

----------------------------------

Since this is a simple room, we will go through the questions and I will show my thought process and work to solve each question.

# How many services are running under port 1000?

To figure this out, we need to run a specific nmap command:

![nmap](./screenshots/nmap.png)

Lets go over this command:

`sudo nmap 10.10.127.163 -p1-1000 -T5 -oN nmap.txt`

* `sudo nmap 10.10.127.163` - Runs nmap as a superuser on IP 10.10.127.163 (this was the IP of the box I needed to get into)

* `-p1-1000` - Scan ports 1-1000 (the first 1000 ports)

* `-T5` - Run the fastest scan you can (1 is slowest, 5 is fastest)

* `-oN nmap.txt` - Put normal-output in the file *nmap.txt*

**We see that there are 2 ports open in the first 1000: port 21 and port 80!**

---

# What is running on the higher port?

We only see 2 ports with our small 1-1000 scan, but if we run a larger scan we may see more information.

![big nmap](./screenshots/nmapbig.png)

Running our scan with `-sC` and `-sV` we see the addition of a port 2222 running **ssh**
---

# What's the CVE you're using against the application? 

If we go to the ip address on a web browser, we are just greeted with the default Apache2 page.

![home](./screenshots/defaultpage.png)

Running gobuster reveals */simple*. Lets check it out.

![simple](./screenshots/simple.png)

This is more interesting! Exploring the page also reveals the version of CMS:

![simple version](./screenshots/simple_version.png)

Let's search this version on ExploitDB to see if it reveals anything.

Searchsploit is a kali command that can search for exploits found on exploitDB. We searched for the keywords *cms*, *2.2.8*, *made*, and *simple*. It revealed an SQLI:

![searchsploit](./screenshots/searchsploit_results.png)

Running the same command with the `-w` argument will also show the exploit-db url, which will reveal the CVE number:

![cve](./screenshots/cve.png)

Looks like we are using **CVE-2019-9053**.

---

# To what kind of vulnerability is the application vulnerable?

We know this is a **SQLI** or an *sql injection*.

---

# What's the password?

To use the exploit, lets copy it into the current working directory. (I recommend making a separate directory for each TryHackMe room you do.)

Our original searchsploit result gave part of the path: *php/webapps/46635.py*.

We can run the following command to copy it to your current working directory:

`sudo cp /usr/share/exploitdb/exploits/php/webapps/46635.py exploit.py`

The exploit database is kept in /usr/share for kali linux users, we know it is an exploit by our searchsploit results, and the rest of the path was given to us. I saved it as *exploit.py* in my directory.

Now lets try running the exploit:

![exploit](./screenshots/exploit.png)

If you type in `python2 ./exploit.py`, it will give you examples usage of the exploit, which is how I knew what command to run. 

We see that the password is **secret**

---

# Where can you login with the details obtained?

We can log into the FTP server as anonymous and download the only file in it. It contains nothing important, so we don't need the login details there.

Trying the ssh we need a username and password, lets try Mitch and our newly obtained password.

`ssh mitch@10.10.127.163 -p 2222` - remember we have to use port 2222 because ssh is not running on port 22 on this box. Using the password we got earlier, we can successfully log in. 

![ssh](./screenshots/ssh.png)

That shows us that the password can be used with **ssh**.

---

# What's the user flag?

We can see one file in the directory we start in, which is Mitch's home directory. Reading the file gives us the flag: **G00d j0b, keep up!**

![user flag](./screenshots/ssh2.png)

---

# Is there any other user in the home directory? What's its name?

If you do `ls /home`, you can see another user named: **sunbath**

---

# What can you leverage to spawn a privileged shell?

If you run the command `root -l`, it will show you what the user can run as root, and whether it requires a password:

![sudo -l command](./screenshots/root_l.png)

This shows us that Mitch may run /usr/bin/vim (**Vim**) without a password. 

---

# What's the root flag?

Head over to [gtfobins](https://gtfobins.github.io/), a great list of Unix binaries that can be used for privilege escalation. Type "vim" into the searchbar (because we know vim can be run as super user) and click on "Sudo". 

![vim sudo](./screenshots/gtfo.png)

Run this command on the ssh shell and you will become root:

![becoming root](./screenshots/become_root.png)

Congrats! You are now root simply because of a misconfigured vim. 

![root](./screenshots/root_flag.png)

The flag is **W3ll d0n3. You made it!**! 

Happy hacking :)







# EasyCTF

_Author: Minimcloving_
_Date: 23 July 2021_


## Enumeration

First lets run an `nmap` scan against the IP address to see what ports are open and what OS is running.

![](./screenshots/nmap.png)

Lets break down our command:
`nmap -sC -sV -oA nmap_init.txt 10.10.9.12`

* `nmap` = command to run an nmap scan
* `-sC` = enables the most common scripts to be run during your scan
* `-sV` = run version detection to get the version of some services
* `-oA` = output in the three major formats (normal, XML, Grepable) at once
* `nmap_init.txt` = filename for the output. Will be appended by .gnmap, .nmap, or.xml. The `.txt` is not needed, and you should actually avoid putting it in your filename variable since the name will be appended anyway.
* `10.10.9.12` = the IP address of the box you are trying to scan

Now, lets break down our nmap scan results

* We have several ports open: 21, 80, and 2222.
	* 21 is an ftp server
		* ftp-anon tells us that you can login as `anonymous`
	* 80 is a http server
	* 2222 is SSH

Knowing there is a webapp, lets run `gobuster` right away to give it time to crack

This is the command we run:

![](./screenshots/gobuster_command.png)

Lets break down the command:

* `gobuster` = run the gobuster command
* `dir` = uses directory/file enumeration mode (since we are trying to enumerate directories on the webapp, this is what we want to use)
* `-u http://10.10.9.12` = specifies the target url we want to scan 
* `-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` = this one will take a little explanation
	* `-w` specifies the wordlist. This is what dirbuster will run through to find possible directories that work in the webapp
	* `/usr/share/wordlists/` this the file default file path for wordlists that come included in Kali Linux distributions. These wordlists are used for cracking passwords or finding directories, and can be used for a lot more things such as fuzzing. Feel free to read up on what wordlists are used for online.
	* `..../dirbuster/directory-list-2.3-medium.txt` specifies which wordlist you want to use
		* `/dirbuster/` specifies wordlists that were used with the dirbuster command (a common directory enumeration service)
		* `/directory-list-2.3-medium.txt` is the specific, chosen wordlist you want to use. This is a good wordlist to use during CTF directory busting. They also have a _small_ version which is good to use for CTFs too.
* `| tee gobuster.txt`
	* `|` = pipes the results of our command into another command, in this case it is the `tee` command
	* `tee` = this command reads from our pipe (`|`) and displays the output both on the screen (or, what is called _standard output_) and saves the output in a file
		* _Note_: This is a great practice to do with gobuster command because you want to be able to see your results as they come up, but this also allows you to read your results later if your terminal crashes or if you are doing a post-mortem writeup like me :)
	* `gobuster.txt` = simple the name of the file I save the output to

![](./screenshots/gobuster_results.png)

Now lets go through the results:

* The first portion of this (anything with a [+] before it) is just the information we fed into gobuster
* `/simple` & `/server-status` = The directories we managed to find in however much time we let gobuster run (in my case once I found these two I stopped it, but you should usually just let it run in another tab/window)


## Exploring the FTP server

We know there is an ftp server, lets explore it a little bit

