---
layout: post
title:  "Provisioning Hacker & IR builds"
description: "A highlight of the attack & response VM provisioning scripts from Fire-eye"
date:   2019-03-28 14:49:33 +0000
tags: [Malware, Incident Response, Red-Team, Pentesting, Offensive, Defensive]
---

![commando-vm](/blog/assets/Commando.png)

## Intro
As someone whom in the past has been responsible for the consistency of build images for both Pentesting and Incident Response Teams
within medium to large corporation, Netscylla thought that today we would highlight the advantages of two build/provisioning scripts
released by Mandiant's Fire-eye Team:
 * [Commando-VM](https://github.com/fireeye/commando-vm)
 * [Flare-VM](https://github.com/fireeye/flare-vm)
 
## Why Use Provisioning Scripts
Provisioning scripts ensure that all consultants are using approved and consistent builds.  That they are using the same software; either open-source
or with a commercial license. That all software is: 
 * using the appropriate licensed versions,
 * patched and up-to-date,
 * configured consistently,
 * all staff are trained using the same tools and methodologies,
 * and that staff performing their job in a similar consistent way.

Ok, we admit on occasion there can be some configuration drift, if consultants are using their own laptops and not issuing the update command. But this can be easily resolved!

Additionally, clients can sometimes be concerned about foreign laptops / virtual machines on their network. These build scripts make it easier to share with client's
so that they can provision most frequently used tools into their environments. 

For example over the last few years, and with the recent migration to 
cloud environments it has often been much easier to request a copy of [https://www.kali.org](Kali Linux) in the environment. It has a majority of the common
hacking tools for pentesting, and life becomes much easier as opposed to arguing with system administrators, trying to get your builds or hacking-tools into
the environment. But there has been no real option for consultants that prefer Windows, or for Consultants that are restricted to operate in Microsoft Windows-Only environments.

## But what about Windows Tools?
Up until recently there have been no real attack / response virtual-machines or provisioning scripts that make the deployment of Windows hacking/response tools easy to install.

We have been limited to the following choice:
 * Tools and licenses loaded on DVDs
 * Tools and licenses loaded on USB sticks/drives 
 * Large virtualised images loaded via USB drives
 * Custom scripts (.bat, .vbs, .ps1)
 * Experimental use with [Ansible](https://www.ansible.com) and [Chocolatey](https://chocolatey.org)
 
## Why use the Fire-eye scripts
Mandiant Fire-eye, uses [Chocolatey](https://chocolatey.org)! A tool we are already familiar with to load several software packages that are often used in our line of work.
The beauty of sharing these scripts with the community, is that it is easy to fork on [Github.com/fireeye](https://github.com/fireeye/), amend and insert any extra packages or 3rd party software that
you like to use within your company: e.g. insert custom scripts, and any in-house software.

To modify any packages already in the Commando-VM resposiorty its as easy as amending the Win10 install tool script packages .json file
 * [packages.json](https://github.com/fireeye/commando-vm/blob/master/commandovm.win10.installer.fireeye/tools/packages.json)

In Flare-vm this is:
 * [packages.json](https://github.com/fireeye/flare-vm/blob/master/flarevm.installer.flare/tools/packages.json) 

## What about the base OS?
This can be configured by either of the following orchestration software:
 * [Terraform](https://www.terraform.io)
 * [Packer](https://www.packer.io)
 * [Vagrant](https://www.vagrantup.com)
 
## Commando Installation
Create and configure a new Windows Virtual Machine
Ensure VM is updated completely. You may have to check for updates, reboot, and check again until no more remain
Take a snapshot of your machine!
Download and copy install.ps1 on your newly configured machine.
Open PowerShell as an Administrator
Enable script execution by running the following command:
```
Set-ExecutionPolicy Unrestricted
.\install.ps1
```
You can also pass your password as an argument: 
```
.\install.ps1 -password <password>
```
### Adding and updating packages
As commando hooks chocolatey, its as easy as
```
 cinst notepadplusplus
```
or
```
 choco install notepadplusplus 
```
update all packages
```
 cup
```
## Flare Installation
If you are using PowerShell v2:
```
Set-ExecutionPolicy Unrestricted
iex ((New-Object System.Net.WebClient).DownloadString('http://boxstarter.org/bootstrapper.ps1')); get-boxstarter -Force
```
And PowerShell v3 or newest:
```
Set-ExecutionPolicy Unrestricted
. { iwr -useb http://boxstarter.org/bootstrapper.ps1 } | iex; get-boxstarter -Force
```
Next, you can deploy FLARE VM environment as follows
```
Install-BoxstarterPackage -PackageName https://raw.githubusercontent.com/fireeye/flare-vm/master/install.ps1
```
### Adding and updating packages
As commando hooks chocolatey, its as easy as
```
 cinst notepadplusplus
```
or
```
 choco install notepadplusplus 
```
update all packages
```
 cup
```

## Commando vs Flare
Obviously, these provisioning scripts help install tools for either the red or blue team members - there is some overlap, but we advise running these scripts in different machines or VMs.

**We advise running these scripts in TWO different Virtualised Machines!**

## Conclusion
Not everyone likes running a pre-determined image, some hackers prefer to use their own specific builds and customised software. But considering that
you could be working in a controlled environment, or have a number of new starters or different contractors.  Using provisioning scripts can help to
quickly set-up their test environment, laptops or virtualised machines to minimise down-time.

For testers that have to burn their images, or leave their hard-drives on a custom site. These scripts remove some of the pain of manually installing common software.

It's worth taking a look at...
