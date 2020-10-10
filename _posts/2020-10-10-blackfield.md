---
title: "Blackfield"
date: 10-10-2020
classes: wide
excerpt_separator: <!--more-->
categories:
  - writeups
tags:
  - htb.hard
  - active directory
  - windows
  - ctf
---

## >> INTRO

Welcome, dear reader, to an epic of fantastic proportions - a story of a small-brain pentester trying to tackle his personal Goliath of the times, `BLACKFIELD`. If you haven't tried this box yet, you absolutely should - everything about it was a 5-star experience. It took me the better portion of 10 hours of off-and-on testing to compromise up to `NT AUTHORITY\SYSTEM` and let me be the first to tell you: they payoff was well worth it. It was a slugfest that involved tons of enumeration and finding small holes to navigate through via SMB and RPC. It involved identifying small administrative mistakes that the server had put into place. It even involved a small amount of memory forensics (which I've never even *thought* about learning, up to this point in my hacking career). This box had it all, and overall I was very impressed with the layout maintaining a very logical real-world feel. Let's get to it!
<!--more-->

Warning for all ye who enter - here be spoilers! I *highly* recommend trying this box first and using the forums as if it were release day - reading on will just spoil the fun!
{: .notice--warning}

## >> BLIND LEADING THE BLIND

First things first, let's start with the standard - snag the IP of Blackfield from the HTB web interface and add it to your `/etc/hosts`. Here's how it looks when on the new VIP+ dedicated servers:
```
10.129.1.243  blackfield.htb
```
Doing this can help us if there's any kind of virtual host routing, but that's not really an issue on this box - it's more of a convenience thing because I'm too lazy to try to memorize the new IP structure for dedicated servers 😛 Once we've got that added, let's run an nmap with `sudo nmap -sCV -O -oA tcp-full -p- blackfield.htb`. Breaking the arguments down:
* `-sCV`: This is a concatenation of `-sC` and `-sV`. This runs all of the default scripts and version checking.
* `-O`: Nmap will make an attempt at running operating system checks. This is mostly based on the banner information it receives, so this can be inaccurate at times. Always double-check with information you've verified.
* `-oA`: This tells Nmap to output results in all of its formats - `.xml`, `.gnmap` (greppable), and `.nmap` (pretty version). It takes in the name of the file you want; in this case, it's `tcp-full` so the final results would be `tcp-full.xml`, etc.
* `-p-`: The argument for this is shorthand for `-p1-65535`, which scans all of the possible ports on the box.

With all of that information, let's see the output of the scan.

```
# Nmap 7.80 scan initiated Fri Oct  9 14:41:00 2020 as: nmap -sCV -O -oA tcp-full -p- -v blackfield.htb
Nmap scan report for blackfield.htb (10.129.1.243)
Host is up (0.049s latency).
Not shown: 65527 filtered ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-10-10 02:54:20Z)
135/tcp  open  msrpc         Microsoft Windows RPC
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=10/9%Time=5F80BF59%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
TCP Sequence Prediction: Difficulty=257 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h02m30s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-10-10T02:56:41
|_  start_date: N/A

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Oct  9 14:54:50 2020 -- 1 IP address (1 host up) scanned in 830.79 seconds
```

There's not a ton open on this box - that makes sense, since it's rated hard after all - but what does it do? From the outside, this looks like a domain controller since we see `DNS:53/tcp`, `KERBEROS:88/tcp`, and `LDAP:389/tcp` open. We'll look deeper into this later; for now, we start enumerating LDAP.

In order to query into LDAP, we'll use `ldapsearch`. It allows us to construct LDAP queries more efficiently than if we were to do so by scripting or by hand. We'll start by getting the default namingcontexts: `ldapsearch -LLL -x -H ldap://blackfield.htb -b '' -s base "(namingcontexts=*)" > namingcontexts.txt` The namingcontexts help us figure out the domain that we're currently looking into at the very list. Namingcontexts also contain the information that a machine needs to know when joining the network. The output is shown here:

![]()

Looking through the output, we notice `dc01.blackfield.local` and add it into our `/etc/hosts`; we also take note that we'll be looking into the `BLACKFIELD.LOCAL` domain. It's not much to go off of, so let's pivot to another protocol for enum.

Another quick win for Windows machines lies in SMB - if left open to unauthenticated users, you can potentially score information or even sometimes sensitive files. Start by listing the files with `smbclient -L \\\\blackfield.htb\\ -U ""`. This produces the following 

## >> BOOM, ROASTED!



## >> THIS IS HELPDESK SPEAKING...


## >> ALL YOUR HASH ARE BELONG TO US


## >> BACK, BACK, BACK IT UP


## >> RECAP/LESSONS LEARNED


## >> REFERENCES

