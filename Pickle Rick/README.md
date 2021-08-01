Pickle Rick
===================

> Armand Alvarez | 31 July 2021

> [Room Link](https://tryhackme.com/room/picklerick)

---

Task 1: Pickle Rick

With this box, we are given nothing but an IP address, and 3 questions, each asking the name of 1 ingredient that Rick needs. So lets begin.

## The First Ingredient

Lets start poking at this IP address we have, see what comes up.

Running the nmap command: `nmap -sC -sV -O -oN nmap.txt <IP>` gives us the following

![nmap](./screenshots/nmap)

Lets break this nmap command down:

* `nmap`  - Tells linux to run an nmap scan
* `-sC`  - Runs default scripts 
* `-sV`  - Probes open ports to determine service/version info
* `-O`   - Enables OS detection
* `-oN nmap.txt`  - Outputs the scan in the *normal* format to the file *nmap.txt*
* `10.10.55.186`  - The IP address of our box

The output tells us that there are two services running: SSH and HTTP.

Since we know there is a web service, I will start up gobuster, a tool for enumerating the directories of a website. I will go over the details later, but while that is running in the background, lets take a look at the website.

--- 

![website home](./screenshots/website_home)

There isn't much to it. A picture and some unassuming text. Taking a look at the page source (on firefox, right-click, then select *View Page Source*) we can find a username: `R1ckRul3s`.

![website home viewing source](./screenshots/website_home_element)

Nice! Our first half of a set of credentials (still need the password.) Make sure you write this down for later. 

Lets see if there is a robots.txt page. Oh, I should probably explain what that is. [Google's developers documentation](https://developers.google.com/search/docs/advanced/robots/intro) says that a *robots.txt* page "tells search engine crawlers which URLs the crawler can access on your site. It is used mainly to avoid overloading your site with requests." Cool! So basically, instead of a robot on the internet having to check every single page your website has to see if it can access it, it can look at the robots.txt page to see where it is allowed to go, saving your website the overhead. 

If we navigate to <IP address>/robots.txt on our web browser, all we get is the text: `Wubbalubbadubdub`

Not too interesting, but, like any text that stands out, you should write it down and see if it does anything interesting *later*.

---

Lets go back to our gobuster results.

![gobuster](./screenshots/dirbuster)

2 results, /assets and /server-status, both of which aren't very interesting. I will let you explore them as you wish.

For now, I will explain what gobuster is, the input, and how to interpret the output. Gobuster is a tool that enumerates directories on a web server. Basically, when you are given an IP address, if you type it into the address bar on the top of a web browser, it takes you to that website's home page. There are often times other pages within that web server address, that can be found by putting a `/` character after the IP address (1.2.3.4/robots.txt or 14.32.122.34/assets.) 

`gobuster dir -u http://10.10.74.255 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt | tee gobuster.txt`

Lets break down this command:

* `gobuster`  - the command to run the gobuster program
* `dir`  - Run in directory mode, because we are enumerating web server directories
* `-u http://10.10.74.255`  - Gives the url to gobuster
* `-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`  - Gives the wordlist to run against the url. If you have kali linux, the /usr/share/wordlists directory is an amazing directory that contains a whole bunch of wordlists to use for many different things. The dirbuster/directory-list-2.3-medium.txt is a good one to use for gobuster directory enumeration. 
* `|`  - Pipes the output into the following command
* `tee`  - This command takes the output and both shows it on the terminal screen (which is called standard output) and saves the output to a file of your choice
* `gobuster.txt`  - The name of the file to save the gobuster results to

On the bottom of the gobuster output, you see `/assets` and `/server-status` which are the directories the program found. 

---

I also ran *nikto* on the IP address to look for any known vulnerabilities. The output revealed something interesting...

![nikto](./screenshots/nikto)

An Admin login page called `/login.php`. Lets explore this. 

---

![login](./screenshots/login)

Nice!! A login page. We know from earlier that a valid username should be `R1ckRul3s`. The only other information we have gathered so far was that random `Wubbalubbadubdub` from the robots.txt page. Lets try that out.

---

![command panel](./screenshots/command_panel)

Awesome, it brought us to a new page, *Command Panel*. Given this is a *command* panel, if we try the command `ls`, it outputs a list of files. 

![trying ls command in command panel](./screenshots/command_panel_ls)

Here, we see one file in particular, *Sup3rS3cretPickl3Ingred.txt*. Trying the `cat` command to read it doesn't work, the command has been disabled. Trying `head` also doesn't work, but `less` works!

![less ingredientfile command panel](./screenshots/command_panel_less)

Our first ingredient is `mr. meeseek hair`

---

## The Second Ingredient

Since this is a command panel, and it clearly runs linux commands, lets see if we can run a reverse shell script on it. 

First, open a new tab on your terminal and run the command:

`nc -lvnp 1234`

This tells netcat to:

* `-l`  - listen for inbound connects
* `v`   - verbose 
* `n`   - numeric-only IP addresses, no DNS
* `p`   - specify the port to listen on

So now netcat is listening on port 1234 for connections coming into it. The connection would have to come into our openvpn IP address. 

![nc listening](./screenshots/nc_listen)

We can find our openvpn IP address with the command: `ip a s`

![ip](./screenshots/ip)

The ip address we want to use for our reverse shell is tun0 (in my case, 10.6.77.21)

Going back to the command panel, lets see if we have access to python3. Typing `python3 --version` gives us a response with the version number, so we know we can use python3. Head over to the [pentest monkey cheatsheet](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) to grab the Python reverse shell. (BTW, **pentest monkey** is an amazing resource for spawning a reverse shell if you have command execution vulnerabilities, which is the command panel we have been using.) 

Throw this line of code into sublime or vim to edit it, we need to change what version of python we are using and the IP address:
* Change [python] -> [python3]  - this is because we don't know if we have python, but we know we have python3.
* Change [10.0.0.1] -> [*your tun0 IP address we got before*]  - This sends the connection to that IP address, which netcat is listening to. The port number (1234) can stay the same because netcat is listening to that port number, but if you chose a different one during the netcat setup, make sure you change the port in the reverse shell code to the one you picked.

Throw that new line of code into the command execution panel and run it. Going back to our netcat, we see that we are on!

![netcat connected to reverse shell](./screenshots/nc_connect)

---

Lets explore this shell a little bit. 

Running `whoami` tells us we are *www-data*. `pwd` tells us we are in */var/www/html* We can read the *clue.txt* file and we get back "Look around the file system for the other ingredients" so lets go do that.

Heading into the */home* directory we see two users: *rick* and *ubuntu*
The *rick* directory has just one file: *second ingredient*. Reading it gives us `1 jerry tear`!!!! That is our second ingredient :)

---

## The Third Ingredient

So, we got basically all we need from the Rick directory, what about checking out that /root directory. If you try, you can see that you aren't allowed in. This is because we don't have root privileges to get into that directory. Lets try and change that. 

Typing `sudo -l` will tell us what commands the current user (www-data) is allowed (and forbidden) to run. 

![sudo l](./screenshots/sudo_l) 

The bottom shows that www-data may run `(ALL) NOPASSWD: ALL`, meaning that we can run nearly any command as sudo without a password. Knowing this, typing `sudo -i` will run a new shell as root, giving you that root access that we needed before.

![sudo i](./screenshots/sudo_i)

From here, we can find the flag in the /root directory and read it for the last ingredient: `fleeb juice`

![flag](./screenshots/3rd_ingredient)



---

And that is really all there is! As you can see, sometimes `sudo -l` can reveal that you can run virtually any sudo command, which is a horrible mistake on the side of the security manager, but great for you. Happy hacking!

:)
