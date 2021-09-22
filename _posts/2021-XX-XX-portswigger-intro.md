---
title: "Portswigger Web Security Academy: Introduction"
date: 2021-09-17
classes: wide
excerpt_separator: <!--more-->
categories:
  - tutorials
tags:
  - webapp
  - resources
---

## >> INTRO

Hi all, and welcome to another post series from the number one knucklehead ~~ninja~~ computer hacker! Over the next indeterminate amount of time, I'll be tackling PortSwigger's Web Security Academy in hopes that I'm not trash at my job. The idea here is that I'll give you a small primer on what each vulnerability is and then we'll get cracking on each challenge and how to exploit it. As they say, teaching is the best way to learn you some skills. Sound fun? That's cuz it is, sucka!

![GIF of Mr. T calling people a sucker.](/assets/images/pswa/intro/sucka.gif)

## >> WHAT IS PORTSWIGGER WEB SECURITY ACADEMY?

Starting with the obvious - if you're unaware of who [PortSwigger](https://portswigger.net/about) are and what they do, go check out that link and see for yourself. If you're even remotely interested in bug bounty/pentesting/web application security, their product BurpSuite is going to be an absolute _MUST_ have. The TL;DR is this:

>PortSwigger produces a bunch of research and a web application proxy named BurpSuite.

_Yes, I wrote that - no, it's not on their site anywhere._ Oh well, moving on. A web app proxy is an invaluable tool for the budding bug bounty hunter or pentester as it lets you proxy and intercept _all_ web traffic and mess with the contents. This ends up with a bunch of nifty things you can use it for - modifying parameter values to bypass client-side controls, ~~smoking~~ inspecting forms to reveal hidden ~~laser traps~~ values, and so much more. There are a few versions out there but for everything from here on I'll be using BurpSuite Community Edition, aka free BurpSuite. I'm doing this in hopes that you, the lovely viewer at home, can follow along and re-create everything I do. I'm going to strongly urge that if you plan on doing this full-time though, look into Professional - it removes some rate limiting and unlocks lovely features (like Burp Collaborator client).

Additionally, we'll be hitting all the topics on the [Learning Path](https://portswigger.net/web-security/learning-path) - I'll try to go roughly in order but may jump around due to interest/time, or if I see things that pair nicely together. Here's the list of topics in case you don't want to look at the learning path:

1. SQL Injection
2. Authentication Issues
3. Directory Traversal
4. Command Injection
5. Business Logic Vulnerabilities
6. Information Disclosure
7. Access Control Issues
8. Server-Side Request Forgery (SSRF)
9. XXE Injection
10. Cross-Site Scripting (XSS)
11. Cross-Site Request Forgery (CSRF)
12. Cross-Origin Resource Sharing (CORS)
13. Clickjacking
14. DOM-based Vulnerabilities
15. WebSockets Issues
16. Insecure Deserialization
17. Server-Side Template Injection (SSTI)
18. Web Cache Poisoning
19. HTTP Host Header Attacks
20. HTTP Request Smuggling
21. OAuth Authentication

_Wheeeew, that was a lot to type out._ Although it looks daunting, we can do it, I believe in us. Also, I'll update this post with links to the relevant sub-post once it's published. It'll be like a little living map until the series is doneski.

![GIF of the movie Airplane!, with the character sweating profusely.](/assets/images/pswa/intro/sweat.gif)

## >> OUTRO

Cool, so now you know what's going down. Hopefully this little series will help me to find more bugs at work as well as prove to myself that I'm _not_ stupid and/or awful at hacking things. Hopefully we'll all learn a bunch and be able to tackle all of the labs - until next time, keep on keeping on and happy hacking!

\- sp1icer

## >> RESOURCES

[https://portswigger.net/about](https://portswigger.net/about)  
[https://portswigger.net/web-security/learning-path](https://portswigger.net/web-security/learning-path)  