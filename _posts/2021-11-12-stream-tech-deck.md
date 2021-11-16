---
title: "Do a Kickflip: Stream Deck Automation FTW"
date: 2021-11-12
classes: wide
excerpt_separator: <!--more-->
categories:
  - tutorials
tags:
  - automation
  - vagrant
  - virtualization
---

> All I care about is that people remember me as a good skater, as someone who was innovative.  
\- Tony Hawk

## >> STREAM TECH DECK

Hi all, welcome back to your one-stop-shop for maybe fun, possibly worthless hacking knowledge! In today's episode of `weird stuff sp1icer's done`, we explore how I use my ~~tech deck~~ stream deck to automate things like virtual machine provisioning, setting up for HTB practicing, note taking, and more! Hopefully by the end of this post you should have enough knowledge to create your own commands and come up with creative automation solutions to your problems. Let's get started!

<!--more-->
Note: You'll need to be set up to rock with Vagrant. Need help with Vagrant? I've got a decent primer that I wrote a while back, [go give it a look](https://sp1icer.dev/tutorials/using-vagrant-for-fun-and-profit/).
{: .notice--info}

## >> WHAT IS A STREAM DECK, ANYWAYS?

Starting off with the easy stuff, a Stream Deck is Elgato's all-in-one broadcasting solution that is essentially a _very_ nice macro pad. Here's their paragraph on customizing the Stream Deck:

> Customizing Stream Deck is effortless. Simply drag and drop actions onto keys, and make them your own with custom icons. Need more actions? Turn keys into folders to amass and access as many actions as you want. Better yet, save unique key configurations as dedicated profiles for different games and apps, switch between them on the fly, and share them with fellow creators.

Okay, so that's pretty neat - per-app profiles and some neat integrations mean that we should be able to control OBS as streamers. It's _definitely_ marketed at the streaming crowd - [go check the site if you don't believe me](https://www.elgato.com/en/stream-deck) - but we definitely are able to utilize this thing to perform [system actions](https://help.elgato.com/hc/en-us/articles/360028234471-Elgato-Stream-Deck-System-Actions-Hotkey-File-Open-Weburl-Media-Controls-) to some simple degree. And as if _that_ weren't enough, they've [even provided an SDK](https://developer.elgato.com/documentation/stream-deck/sdk/overview/) that we can use to extend functionality (that's outside the scope of this post, though).

If you poke through the System Actions help article, you'll find an action called `Open` - that's gonna be our best friend in the next few sections. Soon enough we'll have our system doing flips for us:

![A person doing a tre flip on a skateboard.](/assets/images/stream-tech-deck/tre-flip.gif)

## >> DO A KICKFLIP!

So let's start off with something pretty easy - for the first bit we're going to just open programs. Although easy, this is going to be a cornerstone of our process for automating VM deployments. As an example, we'll do my action for opening Obsidian so I can take notes for everything. Start with an open spot in the Stream Deck software and add the `Open` action. Give it whatever `Title` you want and then for the `App / File` section, choose the path to `Obsidian.exe`. It should look something like this: `<Drive Letter>:\Users\<Username>\AppData\Local\Obsidian\Obsidian.exe`. Now test out your new command - click the physical button on your Stream Deck and you should see Obsidian open!

Note that the same process goes for any other program that we want to open; this opens up our second cornerstone. In order to run terminal commands, we can use the same `Open` action with the new Windows Terminal being the target of our affections. Same deal - set it up with the path similar to this: `<Drive Letter>:\Users\<Username>\AppData\Local\Microsoft\WindowsApps\wt.exe`.

And finally, the piece that lets us run commands in series - the `Multi Action`. [Check the docs for multi-action](https://help.elgato.com/hc/en-us/articles/360027960912-Elgato-Stream-Deck-Multi-Actions) and it'll make a bit more sense why I love it so much for automation. It effectively lets us perform multiple actions from one button press, neat! Let's build an action to get set for blog writing:

![Stream Deck software showing a multi-action designed to open Visual Studio Code for blog editing, among other programs.](/assets/images/stream-tech-deck/multi-action-blog-edit.png)

_Ooooookaayyyyyy, that's a pretty steep jump._ It's okay dearest reader, it's less intimidating than it looks. It starts with opening Windows terminal - we went over that already - and then the only other actions it performs are typing, and opening GitHub Desktop for me. Delays are exactly what they sound like - it effectively `sleep(500ms)`'s itself for us, although you can set your preferred duration for the delay. Onto the next pieces...

All I'm doing is having my terminal type `cd <path>` and then there's this nifty piece that hits enter for us:

![Stream Deck software showing that the system will press "Enter" after typing the message.](/assets/images/stream-tech-deck/press-enter.png)

Once we have this in place, we can make Visual Studio Code open the blog for us by the following action:

![Stream Deck software command to open Visual Studio Code for us automatically.](/assets/images/stream-tech-deck/vs-code-open.png)

Finally we open GitHub Desktop at the following path: `<Drive Letter>:\Users\<Username>\AppData\Local\GitHubDesktop\GitHubDesktop.exe`

It's _SERIOUSLY_ that easy.

![A Green Bay Packers football player dusting off his shoulder to show that it was too easy to score the touchdown.](/assets/images/stream-tech-deck/too-easy.gif)

## >> VAGRANT AT ITS FINEST

Aight, we've got the pieces now - let's put them together. You can probably see how to do this by now but here we go anyways... time to build my HacktheBox + vagrant setup. There are 2 multi-actions that go into this one; I've lovingly named them `HTB Up` and `HTB Down`. I'm very creative, you see. Starting with `HTB Up`, the actions are:

```
1. System: Webite
----> URL: https://app.hackthebox.com/machines
2. System: Open
----> Path: <Drive Letter>:\Users\<Username>\AppData\Local\Microsoft\WindowsApps\wt.exe
3. Stream Deck: Delay
----> Time: 1000ms
4. System: Text
----> Text: cd <path_to_vagrantfile>
----> Press Enter: Yes
5. Stream Deck: Delay
----> Time: 1000ms
6. System: Text
----> <Drive Letter>:\HashiCorp\Vagrant\bin\vagrant.exe up
----> Press Enter: Yes
```

So all the actions there should be pretty self-explanatory - the only new one is that we use the `System: Website` command to pop open HTB to the machines tab so that I can lazy-click the machine that I want. I'm very lazy, you see.

The `HTB Down` action is essentially the same thing, just reversed:

```
1. System: Open
----> Path: <Drive Letter>:\Users\<Username>\AppData\Local\Microsoft\WindowsApps\wt.exe
2. Stream Deck: Delay
----> Time: 1000ms
3. System: Text
----> Text: cd <path_to_vagrantfile>
----> Press Enter: Yes
4. Stream Deck: Delay
----> Time: 1000ms
5. System: Text
----> <Drive Letter>:\HashiCorp\Vagrant\bin\vagrant.exe destroy -f
----> Press Enter: Yes
```

And that's it! If you press the `HTB Up`, you get a VM + HTB's machines section popped up for you. Pressing `HTB Down` knocks down the VM (I haven't bothered figuring out if there's a way to close a program on Stream Deck, sorry). I'd imagine that you could do similar with programming, creating all kinds of buttons for compiling and running your program, starting docker containers and opening the web page, et cetera et cetera. Let your imagination run wild, friends!

![Kazoo Kid doing...something. I'm not really sure what.](/assets/images/stream-tech-deck/kazoo-kid.gif)

## >> OUTRO

Cool, now you've got your computer automatin' and hatin', now get to hacking. 

## >> REFERENCES

In the actual post:  
[https://www.elgato.com/en/stream-deck](https://www.elgato.com/en/stream-deck)  
[https://help.elgato.com/hc/en-us/articles/360028234471-Elgato-Stream-Deck-System-Actions-Hotkey-File-Open-Weburl-Media-Controls-](https://help.elgato.com/hc/en-us/articles/360028234471-Elgato-Stream-Deck-System-Actions-Hotkey-File-Open-Weburl-Media-Controls-)  
[https://help.elgato.com/hc/en-us/articles/360027960912-Elgato-Stream-Deck-Multi-Actions](https://help.elgato.com/hc/en-us/articles/360027960912-Elgato-Stream-Deck-Multi-Actions)  


Extra reading:  
[https://sp1icer.dev/tutorials/using-vagrant-for-fun-and-profit/](https://sp1icer.dev/tutorials/using-vagrant-for-fun-and-profit/)