---
title: "Vagrant: For Fun and Profit"
date: 2020-08-11
header:
  image: /assets/images/vagrant-for-fun/vagrant-banner.jpg
categories:
  - infrastructure
  - tutorial
tags:
  - vagrant
  - packer
  - automation
---

## >>INTRO

Have you ever decided to nuke your penetration testing machine because it got too polluted? Maybe you needed multiple configurations for different clients or use cases (such as web app hacking, hardware hacking) that needed specialized tools? Or maybe you're the type of person who needs to quickly initialize a small test network that has a few clients, such as spinning up an instance of [a small Caldera network](https://github.com/mitre/caldera)? If any of these situations sound familiar, it might be time to migrate your workflow over to include [Vagrant](https://www.vagrantup.com/)!

## >>WHAT IS VAGRANT?

Vagrant is a piece of free-and-open-source software (FOSS) that I've been using for around 1.5 years now - and it's absolutely changed how I do things both at work and at home. I'll let Hashicorp explain it themselves:

> Vagrant is a tool for building and managing virtual machine environments in a single workflow. With an easy-to-use workflow and focus on automation, Vagrant lowers development environment setup time, increases production parity, and makes the "works on my machine" excuse a relic of the past.

So ultimately, what does this mean? If you've kept up with the DevOps world of the past few years this should all sound pretty familiar - Vagrant allows us to deploy and manage virtual machines in an automated fashion using Infrastructure-as-Code. Ultimately you end up writing a piece of code - the almighty `Vagrantfile` - and that is what Vagrant reads in to figure out its marching orders.

## >>PENTESTING IN THE STONE AGES

Let's think back to how we would make virtual machines previously - let's assume it's a standard job for a client who has a web application, maybe you're doing some [bug bounty work like Stök](https://www.youtube.com/channel/UCQN2DsjnYH60SFBIA6IkNwg) and wanted a clean environment. You'd first go grab the ISO for the OS distro of your choice - for most people that would be either Kali or Parrot - and then you'd go into your hypervisor and create a new machine and *manually* walk through the install process. You'd finally be dropped in to a terminal or DE and start your customizations - performing things like `sudo apt-get update && sudo apt-get full-upgrade -y` and installing tools like [Comby](https://comby.dev/) and [pwndbg](https://github.com/pwndbg/pwndbg). Once you were finally finished you would snapshot the image or *maybe* you'd do something like linked clones in VMWare to make a pseudo-golden image that you could pass around on engagements. Overall it's not a terrible process - but is there a better way?

## >>THE FUTURE IS NOW

Let's change up our process - Vagrant will shake up a little bit of the monotonous tasks and help us modularize our lives, just a little bit. Here's a typical minimal structure I use for my projects.

```
environments/
    |__ extras/
    |__ packer/
        |__ boxes/
        |__ os_folders/
    |__ scripts/
        |__ os_folder_1/
            |__ specific_build_1/
            |__ comby.sh
        |__ os_folder_2/
            |__ different_script.sh
    |__ vagrant
        |__os_folders/
            |__ Vagrantfile
```

So breaking this down, we have:

* `environments`: The top level folder (organization FTW!)
* `extras`: Any files that are miscellaneous (I use this for things like backgrounds, config files, etc)
* `packer`: This is another tool from Hashicorp, we'll cover what it does soon!
* `scripts`: This is where I house any bash scripts that I've written. Generic, shared ones are hosted in `os_folder` and specific builds are hosted in their own sub-dirs.
* `vagrant`: This holds our different OS builds and their `Vagrantfile`s.

There can be other stuff included as you get more into this - for example, I'm *eventually* going to migrate off of relying on cheap bash scripting tricks and instead migrate to Saltstack so I think I'd add in a `salt/` directory to house the scripts there. Speaking of Saltstack, Vagrant has quite a bit of support for [DevOps configuration management providers](https://www.vagrantup.com/docs) - click the link or look up the Vagrant docs and check out "Provisioners" on the sidebar for more info!

Cool, so we have an idea of how a project is structured and are excited about the future of our personal workflow - what do we need to get started? Depending on your configuration, so of these may vary but here's what I'll use for the rest of the tutorial:

* VMWare Workstation Pro
* [Vagrant](https://vagrantup.com)
* [vagrant-vmware-utility](https://www.vagrantup.com/vmware#buy-now)
* [Packer](https://www.packer.io/)

It's worth noting that the VMWare version of Vagrant costs some money - don't let that deter you. They offer the Virtualbox plugin for free, and it works just the same. All code that I show should be relatively easy to port over to a Virtualbox-based setup. Now we'll start with making a simple box!

## >>WHAT WE'LL BE BAKING TODAY

Let's start with describing our directory structure for this project:

```
environments/
    |__ extras/
    |__ packer/
        |__ boxes/
        |__ ubuntu/
            |__ Vagrantfile.tpl
    |__ scripts/
        |__ ubuntu/
            |__ comby.sh
    |__ vagrant
        |__ ubuntu_vc/
            |__ Vagrantfile
        |__ ubuntu_custom/
            |__ Vagrantfile
```

Essentially, this will end up being a simple Ubuntu box that has downloaded Comby and is ready to use it. The final setup will be hosted [on my GitHub](https://github.com/sp1icer/vagrant-for-fun-and-profit) for you to browse through, but I recommend following along and manually performing this. Also - as you do this, you should include a line in your `.gitignore` to not track any files in `boxes/*`. They're just generically not worth putting into GitHub (and also are a decent size)

## >>MAKING A GOLDEN IMAGE LIKE A CHEATER

Now that we've got our project set up, we'll start making our gold image - as Vagrant would call it, a "base box". There are ~~two~~ three ways to do a base box - Vagrant cloud, creating our own, and using Packer to build one from the ground up. We'll skip that last one and cover the first two because they're easier to get a grasp on.

### $IT'S NOT A COMPUTER, IT'S THE CLOUD!

So Vagrant cloud is essentially a repository for already-created boxes, both in public and private (for money). I'll once again defer to Hashicorp for the better definition:

> Vagrant Cloud serves a public, searchable index of Vagrant boxes. It's easy to find boxes you can use with Vagrant that contain the technologies you need for a Vagrant environment.  
You don't need a Vagrant Cloud account to use public boxes.

![Vagrant cloud interface](../assets/images/vagrant-for-fun/vagrant-cloud.png)

Basically, this service lets us browse boxes that other people have made and just use them - pretty fantastic! I know the more paranoid among you don't trust this - and for good reason, as you don't control the box that's being loaded - so that's why we have the second category. For those who don't mind, instructions are straightforward from this point; click on the box that you're interested in and be greeted with the following screen:

![Vagrant box instructions](../assets/images/vagrant-for-fun/vagrant-cloud-instructions.png)

That screen shows you what Vagrantfile you should use with the box. Navigate to `/vagrant/ubuntu_vc`. You can either copy and paste it into a Vagrantfile, or you can run `vagrant init hashicorp/trusty64`. Once you do that, you'll have a shiny new Vagrantfile in the current directory and can just type `vagrant up` - the box will automagically be created! Simple, short, and sweet - our favorite. We'll get to customizing the Vagrantfile in a few sections, so kick back for now.

![vagrant init, super easy!](../assets/images/vagrant-for-fun/vagrant-init-hashicorp.png)

### $A BOX MADE FOR OUR OWN SPECIAL IMAGE

If you don't trust the Vagrant cloud images, there is an alternative - creating a box by hand. The bad news it's a tedious process, just like the stone ages - the good news is we only have to do the whole thing once. Let's start with a Ubuntu image, since that's what I showed in the Vagrant cloud section.

VERIFY THE INSTRUCTIONS BELOW!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

1. Download your target Ubuntu ISO - in my case, Ubuntu Desktop 20.04.
2. Create a new VM in your hypervisor of choice using said ISO - make sure to create a user of `vagrant` and password of `vagrant`. Also set the HDD to 40GB just in case - we shouldn't even get close to that, but it's a maximum so let's be a bit generous.
3. Once installation is done, run `sudo apt-get update && sudo apt-get -y full-upgrade`.
4. Reboot the VM.
5. `sudo apt-get install -y linux-headers-$(uname -r) build-essential dkms`
6. Reboot again.
7. `sudo apt-get install -y open-vm-tools-desktop fuse`
8. Reboot one more time, just to make sure.
9. `mkdir -p /home/vagrant/.ssh/`
10. Download the Vagrant default insecure key: `wget -O /home/vagrant/.ssh/authorized_keys https://raw.githubusercontent.com/hashicorp/vagrant/master/keys/vagrant.pub`
11. `sudo nano /etc/ssh/sshd_config`
    1. Uncomment and change PasswordAuthentication to no
12.  `sudo service ssh restart`
13.  Finally, shut the box down.


### $ARTISINAL JSON FILES

After this whole process, we have a pretty decent setup for a gold image - I recommend not adding more customization than necessary at this point. Here's where we finally use Packer - it will bundle up the box for us and make it ready to use in Vagrant. To do this, navigate to the `packer/` directory and either make/navigate into the `ubuntu_custom` folder there. We'll need a file to tell Packer what to do and how to handle the data - for this, we use the [Packer vmware-vmx provider.](https://www.packer.io/docs/builders/vmware-vmx.html) This file is going to be a JSON file that holds all changes to make to the base image before packaging it for Vagrant consumption. Name this one something like `vagrant-ubuntu-custom-vmx.json`.

```JSON
{
  "builders": [
    {
      "type": "vmware-vmx",
      "source_path": ">>ENTER THE FILE PATH TO YOUR ubuntu.vmx ON DISK HERE!!!",
      "ssh_username": "vagrant",
      "ssh_password": "vagrant",
      "ssh_timeout": "10000s",
      "shutdown_command": "echo vagrant | sudo shutdown -hP now"
    }
  ],
  "provisioners": [
    {
      "type": "shell",
      "execute_command": "echo 'vagrant' | {{.Vars}} sudo -S bash '{{ .Path }}'",
      "expect_disconnect": true,
      "script": "../../scripts/ubuntu/update.sh",
      "valid_exit_codes": [0,1]
    },
    {
      "type": "shell",
      "execute_command": "echo 'vagrant' | {{.Vars}} sudo -S bash '{{ .Path }}'",
      "expect_disconnect": true,
      "script": "../../scripts/ubuntu/open-vm-tools.sh"
    }, 
    {
      "type": "shell",
      "execute_command": "echo 'vagrant' | {{.Vars}} sudo -S bash '{{ .Path }}'",
      "expect_disconnect": true,
      "scripts": 
      [
        "../../scripts/ubuntu/vagrant.sh",
        "../../scripts/ubuntu/sudoers.sh",
        "../../scripts/ubuntu/python.sh"
      ]
    }
  ],
  "post-processors": [
    {
      "type": "vagrant",
      "vagrantfile_template": "Vagrantfile.tpl",
      "output": "../boxes/{{ user `vm_name` }}.box"
    }
  ]
}
```

**NOTE**: For all the Windows users out there - when you do the file path to your `.vmx` file, DON'T follow the Windows standard of using backslashes. Use forward slashes like you would on Unix hosts.{: .notice--warning}

It also has a corresponding vars file. Name this one `vars-vagrant-ubuntu-custom-vmx.json`.

```JSON
{
    "vm_name": "vagrant-ubuntu-custom",
    "box_name" : "vagrant-ubuntu-custom", 
    "box_desc" : "Vagrant for fun and profit Ubuntu template image."
}
```

Looking back in the `vagrant-ubuntu-custom-vmx.json`, you'll see the line `"output": "../boxes/{{ user 'vm_name' }}.box"`. Anywhere you see the double braces, Packer automatically pulls from the defined variables file. You'll specify this when building by means of a command-line switch called `--var-file`. 

### $TIME TO BE A SCRIPT KIDDIE

The keen-eyed among you have noticed that there are references to scripts in the `scripts/` folder. We've now got to make them. Thankfully, they're extremely short and are standard if you've done quite a bit of Linux admin stuff. In order seen above:

`update.sh:`
 ```bash
#!/bin/bash

apt-get update && apt-get -y full-upgrade
[ -f /var/run/reboot-required ] && reboot -f
 ```

`open-vm-tools.sh:`
```bash
#!/bin/bash -eux

apt-get install -y --reinstall open-vm-tools-desktop fuse
reboot
```

`vagrant.sh:`
```bash
#!/bin/bash -eux

# Add the vagrant insecure pub key
mkdir /home/vagrant/.ssh
wget -O /home/vagrant/.ssh/authorized_keys https://raw.githubusercontent.com/hashicorp/vagrant/master/keys/vagrant.pub
chmod 0700 /home/vagrant/.ssh/
chmod 0600 /home/vagrant/.ssh/authorized_keys
chown -R vagrant:vagrant /home/vagrant/.ssh/

# Password-less sudo for vagrant user
echo 'vagrant ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/vagrant
chmod 0440 /etc/sudoers.d/vagrant

# SSH tweak
echo 'UseDNS no' >> /etc/ssh/sshd_config

systemctl restart sshd
```

`sudoers.sh:`
```bash
#!/bin/bash -eux

echo 'vagrant ALL=(ALL) NOPASSWD:ALL' >/etc/sudoers.d/99_vagrant;
chmod 440 /etc/sudoers.d/99_vagrant;
```

`python.sh:`
```bash
#!/bin/bash -eux

apt-get install python3 python3-pip -y
```

I'm not going to go through and explain these individually, so if you don't understand it's time to practice some Google-fu. One important thing to note, however, is the usage of `expect_disconnect` and `valid_exit_codes` in `vagrant-ubuntu-custom-vmx.json` - these prevent Vagrant from losing connection when services restart and return exit codes different than normal. The exit codes **specifically** are useful with updating, as bash returning that there's nothing to update would make Packer exit thinking something went wrong.

### $PACK IT UP AND GO HOME

Alright, so we've now *finally* made it to the end of the Packer section. Let's issue the final command to receive our shiny, newly-packaged box: `packer build --var-file='vars-vagrant-ubuntu-custom-vmx.json' vagrant-ubuntu-custom-vmx.json`. If all goes according to plan you should see the following below:

INSERT IMAGE HERE

## >>LOOK MA, NO HANDS!

The good news - we're halfway there. The bad news? We're only halfway there. We still have quite a bit of Vagrant to cover, and I'm only scratching the surface. It's okay though - let's start really digging in and customizing our new box.

To start, we're going to examine our Vagrantfile in `vagrant/ubuntu_vc/` - keep in mind that the same changes will go across both versions. There shouldn't be any difference from this point forward.

Starting things off, there's a TON of stuff commented out. I highly recommend reading through it, but one of the first things that we'll need to uncomment is the switch to enable the GUI. It should be under a block that starts with `config.vm.provider "virtualbox"`. Uncomment until the nearest `end` statement, and then we'll make our first change so that we're using "vmware_desktop" instead of "virtualbox". Next, up the memory to 4096 - we'll keep it low for now just so we can test without resource restrictions being an issue. At this point, feel free to remove the remaining commented lines - you should end up with a file *very* similar to this:

```ruby
Vagrant.configure("2") do |config|
    config.vm.box = "vagrant-ubuntu-custom"
    config.vm.provider "virtualbox" do |vb|
      # Display the VirtualBox GUI when booting the machine
      vb.gui = true
    
      # Customize the amount of memory on the VM:
      vb.memory = "4096"
    end
end
```
Go ahead and execute a `vagrant up` from this directory. You should see Vagrant start performing its magic by opening VMWare, then a virtual machine is created and added to your machines list. It then boots it and...that's it. That's because we haven't told Vagrant to provision anything but the VM yet, so let's go ahead and add a script called comby.sh to the `scripts/` directory.

```bash
#!/bin/bash

bash <(curl -sL get.comby.dev)
```

Not very exciting, I know. That script was taken directly off the Comby website. To add it to our `Vagrantfile`, we drop a provision line in like so:

```ruby
Vagrant.configure("2") do |config|
    config.vm.box = "vagrant-ubuntu-custom"
    config.vm.provider "virtualbox" do |vb|
      # Display the VirtualBox GUI when booting the machine
      vb.gui = true
    
      # Customize the amount of memory on the VM:
      vb.memory = "4096"
    end
    config.vm.provision "shell", path: "../../scripts/ubuntu/comby.sh", name: "comby.sh", privileged: false
end
```

Now that we've done that we need to re-test our build. My personal preference is to do a `vagrant destroy -f` to completely remove the box, then just type `vagrant up` again. Now our terminal output should look a little different, showing the Comby install process happening:

INSERT PICTURE OF TERMINAL HERE

## >>NOW FOR MY NEXT TRICK...
(section detailing multi-machine configurations)


## >>RECAP

## >>REFERENCES


