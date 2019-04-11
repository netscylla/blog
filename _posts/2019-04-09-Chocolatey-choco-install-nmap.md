---
layout: post
title:  "Chocolatey: choco install nmap"
description: "An overview of a free Windows package management system"
date:   2019-04-09 14:49:33 +0000
tags: [Malware, Incident Response, Red-Team, Pentesting, Offensive, Defensive]
---

![chocolatey](/blog/assets/choco.png)

## Intro
Our last post introduced [Provisioning Hacker & IR Builds](https://www.netscylla.com/blog/2019/03/28/Provisioning-Hacker-IR-Builds.html), and the Fire-eye
provisioning scripts:
 * [Commando-VM](https://github.com/fireeye/commando-vm)
 * [Flare-VM](https://github.com/fireeye/flare-vm)

These are useful to quickly provision a Windows 7/10 image retrospectively for the red/blue teams with all their familiar tools.

These scripts are built on top of [Chocolatey](https://chocolatey.org), but what is chocolatey, and how do we use it... 
 
## Chocolatey
The package manager for Windows Chocolatey - Software Management Automation
 * Easily manage all aspects of Windows software (installation, configuration, upgrade, and uninstallation). Chocolatey is the most reliable when software is included in the package, but can also easily download resources.
 * Take advantage of PowerShell to provide automated software management instructions and Chocolatey’s built-in module to turn complex tasks into one line function calls!

### Installation
Just open Powershell (as Administrator) and copy and paste the following line:
```
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```
Chocolatey should now be installed

### Example 1: Install nmap
It's this simple
```
choco install nmap
```

![choco install nmap](/blog/assets/choco_install_nmap.png)

Should you wish uninstall nmap
```
choco uninstall nmap
```

### Example 2: List installed and managed packages
We use the list command with the -lo (local) flag
```
choco list -lo
```

![choco list](/blog/assets/choco_list.png)

Here you can see other packages that we've installed like Ghidra, 7zip, ilspy and their dependancies. 

All managed through chocolatey.

### Example 3: Upgrading packages with ease:

```
choco upgrade all
```

![choco upgrade all](/blog/assets/choco_upgrade.png)

Above you can see that we have one package that is newer than the default public repository.

This is because we have installed our own internal chocolatey repository, and added our own packages

## Hosting a Chocolatey 
The main instructions can be found here [chocolatey server](https://chocolatey.org/docs/how-to-host-feed).

### Listing current servers
```
choco source list
```

![choco source list](/blog/assets/choco_source.png)

You can see that we've added our own server choco-ns, hosted internally on one of our test networks.

To add your own sources
```
choco source add -n="my_server" -s="http://my_choco_server/chocolatey/"
choco enable -n="my_server"
```

To remove a server
```
choco disable -n="my_server"
choco remove -n="my_server"
```

### Hosting from a cifs share or github
You dont have to build your own chocolatey server.

Instead you can host the packages folder on a cifs/smb share, or from a git folder

```
git clone https://github.com/netscylla/chocolateypackages
```

#### Install Ghidra

```
cd chocolateypackages/packages/ghidra
choco install ./ghidra.nuspec
```

![choco install Ghidra](/blog/assets/choco_install_ghidra.png)

We have already installed Ghidra! Hence the small error message. 

But this example shows you how easy it can be to manage your own repository, and with a number of scheduled commands (or a script), you can keep your packages on Windows up-to-date.
```
git pull
choco upgrade all
```
