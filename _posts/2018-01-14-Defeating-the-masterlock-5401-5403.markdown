---
layout: post
title:  "Defeating the Masterlock 5401-5403"
date:   2018-01-14 13:49:33 +0000
tags: [lockpicking, pentest, redteam, blueteam]
---
# Defeating the MasterLock 5401–5403
The MasterLock 5401 is a wall-mounted key-safe, but at Netscylla we have also seen these inside corporate offices to store data-centre/office keys, and even store corporate credit cards. This particular design of lock is copied by other manufacturers and the retail price varies from £10–30GBP.

**Warning**: This is NOT a high security lock!

This lock has a particular physical vulnerability that means it can be quickly opened without knowing the key-code. Why post about this particular lock?

Last week we were asked to investigate the the likelihood of some physical corporate credit card borrowing in an office. The client had some attempted items purchased on their cards that was unauthorised and that were luckily also blocked by their bank. But they were confused as to how the cards were abused? The affected cards were newly issued cards, locked in a safe and were due to be distributed to the named company directors later that week — it was all very old and perplexing… As the cards were thought to have been secured in a safe!

Having arrived at the client site, and inspecting where these cards were stored, we immediately knew the vulnerability! It is well known how to unlock these particular key-safes and there are many video tutorials on the internet. For awareness Netscylla covers a very high overview on how these key safes are insecure and how easily they can be attacked below:

# Equipment
Below is a ML-5401 that Netscylla own’s and the Sparrows ‘Ultimate decoder’ ($14.95USD or £15GBP from uklockpickers.co.uk). Instead of the decoder, you could use the metal strip from a car’s windscreen blade and cut it down to a manageable size (depending on the thickness of the metal, it may require some additional filing and polishing).

**Edit**: 2018–01–20 a friend mentioned that an old floppy disk cut up also works wonders!

![](/blog/assets/masterlock_1.jpeg)

# The Attack
How to open the ML-5401:
* Place the Ultra Decoder to the left of the first wheel.
* Slide the Ultra in half-way between the bottom and the middle of the wheel, angled slightly upward from the bottom (see the pic below).
* Turn the wheel until you feel a definitive flat spot.
* Repeat for the other three wheels.
Once all flat spots have been found try the unlock switch; if it does not open, move all wheels down one position and try again.

Example positioning of the ‘Ultimate decoder’:

![](/blog/assets/masterlock_2.jpeg)

# Spotted in the wild
Netscylla have seen these locks used in numerous places: including mounted on the front/back walls/pillars of offices, houses, holiday rental homes, outside/inside data-centre walls, private offices, etc.

# Conclusion
Netscylla would say that the use of these particular key-safes for security is highly risky: that the contents of the key-safe should not be left unattended for extended periods of time, and also the keycode should be regularly changed (not that it matters much as it can be opened in less than 15 seconds).

# Mitigating the risk of attack
When using these key-safes Netscylla recommends that you should consider the safe implementation of Crime Prevention Through Environment Design (CPTED):

* **Lines of Sight** — ensure the safe is visible to more than one individual, or access can clearly be identified on CCTV.
* **Boundaries** — what are the boundaries / concerned areas surrounding the placement of this type of safe.
* **Obstacles** — ensure there are no objects blocking visibility
* **Visibility/Disguise** — Have the key safe, placed inside a larger more secure safe.
* **Management** — Don’t leave sensitive or critical material unattended for long periods of time.

# Disclaimer
Netscylla or its staff cannot be held responsible for any abuse relating from this blog post. This post is to raise awareness in the design and security flaws of the ML-5401 lock. It is illegal to attempt unauthorised access on any lock you do not personally own, or have explicit permission in writing from the owner! If in doubt contact your local authority or local lock-sport community for additional advice.



