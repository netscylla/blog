---
layout: post
title:  "OSINT & Social Engineering the Dangers"
date:   2018-07-02 13:49:33 +0000
tags: [OSINT, pentest, redteam, blueteam]
---
I was sitting at the train station platform waiting for my train, when I overheard two guys behind me talk about their social engineering escapades. This wasn’t a normal conversation about hacking or penetration testing; In my opinion it was more about CEO fraud. The two men were dressed in suits, smart shoes, and expensive looking accessories (watches, pens, etc). The conversation went down the lines of one (being more experienced), giving instructions/ideas/training to the second man.

This is post documents the flow of their communication, and we have added some links and pictures to make their conversation easier to understand.

## LinkedIn
The first man went on about how he loves LinkedIn, that all the information he needs to start with is there, and on any job/con, LinkedIn is his first point of reference. He then described certain search techniques, on evaluating potential targets. One of these was taking advantage of a LinkedIn API I had not come across before:

LinkedIn is a valuable resource for picking targets. Just like google-dorks and researching companies through corporate websites and directory listings, LinkedIn has a special URL where you can quickly identify targets of interest:
```
http://linkedin.com/title/<job-title>-at-<company>/
```
Example:

The following URL would list all directors at example-ltd (fictious company)
```
http://linkedin.com/title/director-at-example-ltd/
```
![](/assets/osint_1.png)

This enables us to quickly identify targets and obtain a brief history of the targets careers. We are then free to view their profile in depth — provided they havnt locked down their profile. However, this is not often the case, easy pickings…

Of course you can use also your standard google dorks such as:
* site:example.com director
* site:linkedin.com example.com director

## Doxing
The two men then proceeded to talk about Doxing.

Having a number of targets the next phase picks at other social media sites. If you can match records on other sites such as Facebook, Instagram, Twitter etc; you can find out more details about the person, hobbies, interests, family, events, trips. We can analyse photos for meta data, or depending on the detail work out where they are or what they are doing.

This is the start of reconnaissance phase know as doxing, all this information is neatly recorded, for future reference. Weak or detail-less profiles may be crossed out for those in favour where we have more details.

For more background on doxing visit: [https://en.wikipedia.org/wiki/Doxing](https://en.wikipedia.org/wiki/Doxing)

The first man then described how he found out information about a board member of a consultancy firm, and through a fake website tricked the person into submitting a password they commonly used. He then discovered and travelled to the person’s home address, broke into their WiFi using the same password (name of the town the person grew up), then he proceeded to brake into several other online accounts using the same password discovered from his original doxing research. Once he compromised their email, he could access additional accounts and eventually found banking details! From there he was able to transfer a few thousand pounds into his account without apparently ever being noticed?

## Ramping Up — Contact
Then the first-man spoke about ramping up, he said many people find this the hardest, if you cant do this, no worries you can still make good money through doxing and selling the relevant background information to others!

This is the phase where you reach out to the target, send a connection request on multiple levels, connect to them on facebook, instagram, linkedin, twitter; to make things easier (as sometimes people are wary of strangers) send a polite note with the connection request on how you admire them, that their full of interesting and insightful ideas and want to follow their anecdotes and blog posts. If you don’t have a response by the end of the week, try again the following Monday morning as statistically people are more likely to respond on a Monday morning, than they are on a late Friday afternoon.

If at first you don’t succeed… Try, Try again
He also stated that if at first you dont succeed try again. But don’t appear like a pyscho messaging them everyday, limit it to bi-weekly/monthly. Alter you message to incorporate what you’ve viewed on their other social profiles, if they appear interested in football, ask or insert a punchy one-liner about their team or latest match. Obviously adapt the subject around their interests and hobbies.

Basically in this phase you have to be patient and persistent. Hopefully, if your lucky or a real smooth talker/writer you can get them to connect to you, and hopefully, gain additional access to their profiles and complete a fuller picture of their identity or doxing profile.

## The Attack
### The Fake Award Ceremony
Next phase is the attack, after weeks or months of befriending and small-talk you should have build up a rapport with the target. State that your part of a small private awards company, and that the target has been selected for being outstanding in their field, and would they be able to attend the ceremony to collect the award. — Usually the answer is Yes!

This is where depending how good you are, how much money you can get out of them….
* You can charge them £100 per seat at a table (including dinner), so if they want to bring a partner — thats £200 made easy.
* You can hint to whether themselves or their company may want to sponsor the event, and earn a few thousand!
* And of course if your targeting multiple people in the same industry and sectors, you now have multipliers.

Don’t forget to buy an appropriate cheap domain, host a cheap website, or clone another awards website. Send the target a mail from the fake domain, and send them some fake tickets, an invoice for the tickets, a pdf on whether they want to sponsor the event and bank/card details to collect. You should be quids in by the end of the week.

## The Fake Invoice
The second guy asked about fake invoice scams. The first guy replied about the fact that they are commonly used, and that due to awareness people are more suspicious and unlikely to fall for such scams these days. But he ran through a simple process:
* You have their details,
* you need to know who is their CFO
* use an online re-mailer to mask your real identity
* request that an urgent invoice is paid on the targets behalf.

![](/assets/osint_2.png)

## The End
At this point out train was on the platform, we boarded, but unfortunately the two men did not! So our insight into their experience/scams/consultancy services comes to an end.

## Closing Remarks
These guys were not hackers or computer nerds, typically associated with OSINT and recent cyber social engineering attacks. Instead, they were two guys with average computer literacy (possibly from a consultancy service) discussing how social engineering and using common and easily available tools (such as linkedIn, Google), together with practice, patience and persistence can potentially net scammers a tidy sum of cash.

## Takeaways
* Don’t connect to people you don’t recognise on LinkedIn
* Try to minimise the accessibility of your online profiles, by enforcing a number of ‘Privacy Controls’
* Don’t publish links between your different social media accounts
* Use a different passwords for different sites/services — consider using a password manager!
