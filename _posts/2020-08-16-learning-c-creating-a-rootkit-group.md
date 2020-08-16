---
title: "Learning C: Creating a Rootkit with Friends"
date: 2020-08-16
classes: wide
categories:
  - tutorials
tags:
  - programming
  - malware
  - rootkit
  - group
---

## >> INTRO


## >> ROOTKIT FUNCTIONALITY

The cornerstone of a successful project is defining what success looks like for the assignment. I'm going to break this down into three phases: **minimum viable product, essentials,** and **nice-to-haves**. These phases represent just how much functionality the program will have at different points in time, how robust the program is, and other factors like how well it hides from the prying eyes of an analyst.

`Minimum Viable Product:`
* Hooks an API call
* Modifies the API call on-the-fly
* Returns a result that's different than the original

`Essentials:`
* Enables backdoor functionality to the compromised victim
* Hides directories/files from *one* API call
* Hides network information from *one* API call

`Nice-to-haves:`
* Authentication for backdoor
* Encrypted backdoor communication
* Command-and-control functionality
* Hides directories/files from multiple API calls
* Hides processes from API calls
* Hides network information from tools such as ss, netstat, etc.
* Network propogation
* Keylogging

Creating these lists also help us with two fringe benefits - they help project management, as well as help fighting off scope creep.

## >> PROJECT CONSTRAINTS

Ah yes - everybody's *least* favorite section. Nobody likes being told that they can't do something, which is exactly what this section did to my project. In my case, there were very few restrictions; the most notable is that since I have a regular full-time job and plenty of other side projects (like keeping up with HacktheBox) I ran into time restraints, making the project take quite a bit longer than I had hoped.

## >> PROJECT TIMELINE

## >> GROUP RESPONSIBILITIES

## >> SETTING UP THE SIMULATION

So this section requires some knowledge of Vagrant and how virtual machines work - [go read my post on my Vagrant workflow if you haven't](https://sp1icer.dev/infrastructure/using-vagrant-for-fun-and-profit/) - and then come back here because you're going to need to know how to do multi-machine configurations to understand this next part.

### $PHASE 1: SINGLE MACHINE

### $PHASE 2: FULL NETWORK
Anyone who knows me in real life probably knows one thing about me - I'm a perfectionist. That drive to be perfect *also* extends to my simulations, as it were; I can't leave any stone un-turned. In this case, I decided to model a small architecture that had 2 parts to it - an "attacker" network and a "public" network. This allowed me to set up my command-and-control in the rootkit in a somewhat accurate fashion.

## >> RETROSPECTIVE

Overall, fun

\- sp1icer

## >> REFERENCES

h0mbre's Learning C GitHub Repo: https://github.com/h0mbre/Learning-C  
Too many `man` pages to list  