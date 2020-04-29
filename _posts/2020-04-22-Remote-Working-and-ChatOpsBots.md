---
layout: post
title:  "Remote Working & ChatOps Bots"
description: "A brief post on ChatOps and the use of Bots"
date:   2020-04-22 13:00:01 +0000
tags: [Blue-Team, Red-Team, Pentest, DevSecOps, Threat Intelligence]
---

![ChatOps](/blog/assets/rocketchat.png)

## Introduction
With the Covid-19 pandemic continuing, and remote-working becoming the new norm, many collegues have rasied concerns on shadowing junior hires, and remote collaboration. Our previous post covered the installation of an on-premises chatroom solution to help Team-mates feel more connected and engaged.  

In this post Netscylla will demo the use of **Bots** with [Rocket.Chat](https://rocket.chat) to automate some repeatative tasks, and the use of chatroom channels to oversee and track the developement and progress of junior team members new to the cyber/information security industry.

## One of our bots - the intel_bot

### Our example chat script

Using a bot to perform simple command line actions, like reverse DNS:

![example reverse dns](/blog/assets/TI-job1.png)

Scraping whois information

![example whois](/blog/assets/TI-job2.png)

Utilise the Shodan API to get information on accessible services:

![example shodan](/blog/assets/TI-job3.png)

Take a screenshot of a webpage for evidence purposes:

![taking a screenshot](/blog/assets/TI-job4.png)

The image zoomed in to fit the screen width:

![screenshot full zoom](/blog/assets/TI-job4a.png)

## How to make a bot?

Our bot was written in python and utilised the following public API's from github:
 * [https://github.com/jadolg/rocketchat_API](https://github.com/jadolg/rocketchat_API)
 * [https://github.com/jadolg/RocketChatBot](https://github.com/jadolg/RocketChatBot)

@Jadolg had a simple example of creating a bot that echo's what a user types.  Python is an easy an adpatable programming language to use, and we encourage all of our consultants to attempt learning this specific lanuage for the purposes of scripting in the information security sector.

### So you want to see our bot code?

Here is a snippet of our main bot-code, hopefully it will help you to consider developing your own bots:
```
from requests import sessions
from pprint import pprint
from rocketchat_API.rocketchat import RocketChat
import intel

from RocketChatBot import RocketChatBot

botname = 'intel_bot'
server_url = 'https://your_chat_server_domain'
channel='general'

def printhelp(msg, user, channel_id):
    bot.send_message('My commands are:\nrdns <ip> - perform reverse dns\nwhois <domain> - perform whois\nshodan_ip - retrieve shodan info on a single ip\nsafe2browse <url> - check url with Google\'s safe to browse api\nurlhaus <url/domain> - check domain/url on URLhaus\ndmarc <domain>- print a domains _dmarc DNS record\nss <http(s) url> - screenshot a url', channel_id)

def greet(msg, user, channel_id):
    bot.send_message('hello @' + user, channel_id)
 
def rdns(msg, user, channel_id):
    bot.send_message(intel.reverse_dns(msg), channel_id)

def whois(msg, user, channel_id):
    answer=intel.whois_domain(msg)
    bot.send_message(answer, channel_id)

def shodan_ip(msg, user, channel_id):
    bot.send_message(intel.shodan_address(msg), channel_id)

def safe2browse(msg, user, channel_id):
    bot.send_message(intel.safebrowsing(msg), channel_id)

def urlhaus_check(msg, user, channel_id):
    bot.send_message(intel.check_urlhaus_url(msg), channel_id)

def dmarc_check(msg, user, channel_id):
    bot.send_message(intel.check_dmarc(msg), channel_id)

def getscreenshot(msg, user, channel_id):
    bot.send_message(intel.screenshot_web(msg), channel_id)

bot = RocketChatBot(botname, botpassword, server_url)
#bot.send_message('starting bot...', channel_id='general')
bot.add_dm_handler(['help', ], printhelp)
bot.add_dm_handler(['hey', 'hello', ], greet)
bot.add_dm_handler(['reverse_dns', 'rdns', 'reversedns', ], rdns)
bot.add_dm_handler(['whois', ], whois)
bot.add_dm_handler(['shodan', ], shodan_ip)
bot.add_dm_handler(['safe2browse','safebrowsing', ], safe2browse)
bot.add_dm_handler(['urlhaus','is_phishing', ], urlhaus_check)
bot.add_dm_handler(['dmarc','dmarc_check','check_dmarc', ], dmarc_check)
bot.add_dm_handler(['ss','screenshot',], getscreenshot)
bot.run()
```
## Conclusion

ChatOps bots can be used to automate repeatative tasks, it permits senior team members to observe the progress of junior team members (aka shadowing). It also helps make the team feel more engaged where staff and ask each other questions or even challenges.  There are also many different marketplace plugins for Rocket.Chat to make the chatrooms/channels a fun and engaging place to be.

Have fun developing your own bots...
