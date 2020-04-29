---
layout: post
title:  "Remote Working & ChatOps Video Calls"
description: "A brief post on ChatOps and the use of Video calls"
date:   2020-04-28 13:00:01 +0000
tags: [Blue-Team, Red-Team, Pentest, DevSecOps, Threat Intelligence]
---

![ChatOps](/assets/rocketchat.png)

## Introduction
With the Covid-19 pandemic continuing, and remote-working becoming the new norm, many collegues have rasied concerns on shadowing junior hires, and remote collaboration. Our previous post covered the installation of an on-premises chatroom solution to help Team-mates feel more connected and engaged.  

In this post Netscylla will highlight Rocket.Chats abililty to make video calls, and walkthrough the instructions of creating your own on-premise videobridge should you choose to to host your own solution.

## Video Calling in Rocket.Chat

### Rocket.Chat WebRTC/Meet 
WebRTC or Meet is Rocket.Chat's own take on implementing pure WebRTC. It still a work in progress but if you want to use it you can go to 
```
Administration -> WebRTC.
```
From there you can set where video conferences can be held (channels, private rooms and direct messages).

You can also define a list of STUN/TURN Servers to be used.  By default Google's STUN servers are used.

### Jitsi Meet
You can use the Jitsi Meet video conferencing platform embedded in Rocket.Chat.

The Jitsi Meet project (Jitsi Video Bridge) is a tried and true bandwidth efficient WebRTC compatible SFU (server based) solution from our gracious FOSS partner, Jitsi.

Through the collaboration arrangement with Jitsi, Rocket.Chat users can enjoy reliable and robust group video chat, audio chat, and screen sharing experience out of the box.

To enable Jitsi:
```
Administration -> SETTINGS -> Video Conference 
Then set Enabled to True.
```
From there you can set where video conferences can be held (channels, private rooms and direct messages).

## Installing a Jitsi Videobridge on Ubuntu

### One-Click Install

If you plan on installing Jitsi Meet within your cloud platform, check with your provider as many Cloud Providers support one click installs. Otherwise follow the manual instructions below.

### Set a hostname

First, set the system’s hostname to the domain name that you will use for your Jitsi instance. The following command will set the current hostname and modify the /etc/hostname that holds the system’s hostname between reboots:
```
sudo hostnamectl set-hostname jitsi.your-domain
```
The command that you ran breaks down as follows:

**hostnamectl** is a utility from the systemd tool suite to manage the system hostname.

Check that this was successful by running the following:
```
hostname
```
This will return the hostname you set with the hostnamectl command:
```
jitsi.your-domain
```
Next, you will set a local mapping of the server’s hostname to the loopback IP address, 127.0.0.1. Do this by opening the /etc/hosts file with a text editor:

sudo nano /etc/hosts
Then, add the following line:
```
127.0.0.1 jitsi.your-domain
```
Mapping your Jitsi Meet server’s domain name to 127.0.0.1 allows your Jitsi Meet server to use several networked processes that accept local connections from each other on the 127.0.0.1 IP address. These connections are authenticated and encrypted with a TLS certificate, which is registered to your domain name. Locally mapping the domain name to 127.0.0.1 makes it possible to use the TLS certificate for these local network connections.

Save and exit your file.

Your server now has the hostname that Jitsi requires for installation. In the next step, you will open the firewall ports that are needed by Jitsi and the TLS certificate installer.

### Configuring the UFW Firewall
When you followed the Initial Server Setup with Ubuntu 18.04 guide you enabled the UFW firewall and opened the SSH port. The Jitsi server needs some ports opened so that it can communicate with the call clients. Also, the TLS installation process needs to have a port open so that it can authenticate the certificate request.

The ports that you will open are the following:
 * 80/tcp used in the TLS certificate request.
 * 443/tcp used for the conference room creation web page.
 * 4443/tcp,10000/udp used to transmit and receive the encrypted call traffic.

Run the following ufw commands to open these ports:
```
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 4443/tcp
sudo ufw allow 10000/udp
```
Check that they were all added with the ufw status command:
```
sudo ufw status
```
You will see the following output if these ports are open:
```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
4443/tcp                   ALLOW       Anywhere
10000/udp                  ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)
4443/tcp (v6)              ALLOW       Anywhere (v6)
10000/udp (v6)             ALLOW       Anywhere (v6)
```
The server is now ready for the Jitsi installation, which you will complete in the next step.

### Installing Jitsi Meet
In this step, you will add the Jitsi stable repository to your server and then install the Jitsi Meet package from that repository. This will ensure that you are always running the latest stable Jitsi Meet package.

First, download the Jitsi GPG key with the wget downloading utility:
```
wget https://download.jitsi.org/jitsi-key.gpg.key
```
The apt package manager will use this GPG key to validate the packages that you will download from the Jitsi repository.

Next, add the GPG key you downloaded to apt’s keyring using the apt-key utility:
```
sudo apt-key add jitsi-key.gpg.key
```
You can now delete the GPG key file as it is no longer needed:
```
rm jitsi-key.gpg.key
```
Now, you will add the Jitsi repository to your server by creating a new source file that contains the Jitsi repository. Open and create the new file with your editor:
```
sudo nano /etc/apt/sources.list.d/jitsi-stable.list
```
Add this line to the file for the Jitsi repository:
```
deb https://download.jitsi.org stable/
```
Save and exit your editor.

Finally, perform a system update to collect the package list from the Jitsi repository and then install the jitsi-meet package:
```
sudo apt update
sudo apt install jitsi-meet
```

### SSL Support

First, add the Certbot repository to your system to ensure that you have the latest version of Certbot. Run the following command to add the new repository and update your system:
```
sudo add-apt-repository ppa:certbot/certbot
```
Next, install the certbot package:
```
sudo apt install certbot
```
Your server is now ready to run the TLS certificate installation program provided by Jitsi Meet:
```
sudo /usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```

### Locking Conference Creation
In this step, you will configure your Jitsi Meet server to only allow registered users to create conference rooms. The files that you will edit were generated by the installer and are configured with your domain name.

The variable your_domain will be used in place of a domain name in the following examples.

First, open sudo nano /etc/prosody/conf.avail/your_domain.cfg.lua with a text editor:
```
sudo nano /etc/prosody/conf.avail/your_domain.cfg.lua
```
Edit this line:
```
...
        authentication = "anonymous"
...
```
To the following:
```
...
        authentication = "internal_plain"
...
```
This configuration tells Jitsi Meet to force username and password authentication before allowing conference room creation by a new visitor.

Then, in the same file, add the following section to the end of the file:

*/etc/prosody/conf.avail/your_domain.cfg.lua:*
```
...
VirtualHost "guest.your_domain"
    authentication = "anonymous"
    c2s_require_encryption = false
```
This configuration allows anonymous users to join conference rooms that were created by an authenticated user. However, the guest must have a unique address and an optional password for the room to enter it.

Here, you added guest. to the front of your domain name. For example, for jitsi.your-domain you would put guest.jitsi.your-domain. The guest. hostname is only used internally by Jitsi Meet. You will never enter it into a browser or need to create a DNS record for it.

Open another configuration file at /etc/jitsi/meet/your_domain-config.js with a text editor:
```
sudo nano /etc/jitsi/meet/your_domain-config.js
```
Edit this line:
```
...
        // anonymousdomain: 'guest.example.com',
...
```
To the following:
```
...
        anonymousdomain: 'guest.your_domain',
...
```
Again, by using the guest.your_domain hostname that you used earlier this configuration tells Jitsi Meet what internal hostname to use for the un-authenticated guests.

Next, open */etc/jitsi/jicofo/sip-communicator.properties*:
```
sudo nano /etc/jitsi/jicofo/sip-communicator.properties
```
And add the following line to complete the configuration changes:
```
org.jitsi.jicofo.auth.URL=XMPP:your_domain
```
This configuration points one of the Jitsi Meet processes to the local server that performs the user authentication that is now required.

Your Jitsi Meet instance is now configured so that only registered users can create conference rooms. After a conference room is created, anyone can join it without needing to be a registered user. All they will need is the unique conference room address and an optional password set by the room’s creator.

Now that Jitsi Meet is configured to require authenticated users for room creation you need to register these users and their passwords. You will use the prosodyctl utility to do this.

Run the following command to add a user to your server:
```
sudo prosodyctl register user your_domain password
```
The user that you add here is not a system user. They will only be able to create a conference room and are not able to log in to your server via SSH.

Finally, restart the Jitsi Meet processes to load the new configuration:
```
sudo systemctl restart prosody.service
sudo systemctl restart jicofo.service
sudo systemctl restart jitsi-videobridge2.service
```
The Jitsi Meet instance will now request a username and password with a dialog box when a conference room is created.

Your Jitsi Meet server is now set up and securely configured.

