---
title: "Learning C: Creating a LDPRELOAD Rootkit"
date: 2020-08-16
classes: wide
categories:
  - tutorials
tags:
  - programming
  - malware
  - rootkit
---

## >> INTRO

Recently, I embarked on one of my most interesting learning journies to date - I took on [h0mbre's Learning C repository](https://github.com/h0mbre/Learning-C) and struggled my way through learning to code in C. Before I get too far into the article, make sure to check out the GitHub - seriously, I can't speak highly enough of it, it's fantastic work. The GitHub is a fantastic jumping off point for anyone who wants to learn and is like me - that is, needs a project-based curriculum rather than sporadic assignments that don't relate to anything. The course starts you off by learning your standard "Hello World" programs and then teaaches you various building blocks while it eventually leads you towards creating your very own, bespoke LDPRELOAD-based userland rootkit. This post is my *solo* solution to "Assignment 28: Re-creating the Jynx userland rootkit" - I'm working on leading a group effort to write a robust rootkit with a small team. Let's begin!

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

Creating these lists also help us with two fringe benefits - they help project management, as well as help fighting off scope creep.

## >> PROJECT CONSTRAINTS

Ah yes - everybody's *least* favorite section. Nobody likes being told that they can't do something, which is exactly what this section did to my project. In my case, there were very few restrictions; the most notable is that since I have a regular full-time job and plenty of other side projects (like keeping up with HacktheBox) I ran into time restraints, making the project take quite a bit longer than I had hoped.

## >> PROJECT TIMELINE

## >> SETTING UP THE SIMULATION

So this section requires some knowledge of Vagrant and how virtual machines work - [go read my post on my Vagrant workflow if you haven't](https://sp1icer.dev/infrastructure/using-vagrant-for-fun-and-profit/) - and then come back here because you're going to need to know how to do multi-machine configurations to understand this next part.

Anyone who knows me in real life probably knows one thing about me - I'm a perfectionist. That drive to be perfect *also* extends to my simulations, as it were; I can't leave any stone un-turned. In this case, I decided to model a small architecture that had 2 parts to it - an "attacker" network and a "public" network. This allowed me to set up my command-and-control in the rootkit in a somewhat accurate fashion.

## >> COURSE REVIEW

This section isn't really going to pertain to the rootkit itself - I just really want to emphasize that you *absolutely should give this course a try*. Seriously. It's free, it's on the public GitHub, and I think it's particularly well-organized. I had no programming experience in C coming into this, I had never written *ANY* kind of malware before it, and it helped me learn all of that and more. It's worth the time investment.

\- sp1icer

## >> REFERENCES

h0mbre's Learning C GitHub Repo: https://github.com/h0mbre/Learning-C  
Too many `man` pages to list  