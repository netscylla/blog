---
layout: post
title:  "Red-Teaming and the Importance of Failing!"
date:   2018-07-29 13:49:33 +0000
tags: [pentest, redteam, blueteam]
---
![](/assets/alert.png)

In the case of Red Teaming, the mistake is most likely completely harmless, unless it’s one of these:
* An ethics breach
* A trust violation
* Exposure of company, client, customer, or stakeholder data
* Reduction of security controls resulting in a breach
* Serious impact or Denial-of-Service on critical operations.

Any of those are very severe and probably require very severe consequences — large legal expenses, code-of-conduct reviews, followed by reviewing and amending methodologies and checklists, in an attempt to prevent the same error happening again.

**Notice that detection of the Red-Team during a campaign is NOT AN ISSUE!**

Everyone seems to forget that red-teaming is a cyber maturity exercise to determine if the correct detection, response, or attribution mechanisms are in place and sufficient for early detection and remediation against cyber threats.

## Why is Red-Teaming just a game?
Because red teams are simply not real! They are simulated! A game we play to set playbooks, for learning and development. The breaches should be pretend! The consequences are growth, not damage, nobody should be getting hurt or reprimanded!

While we certainly want to maximise the realism when we simulate a threat actor or APT adversary. Our goal is not to win — it’s to improve the detection and response capabilities of the Blue Team. Teammates may even get frustrated for a bit if a cover story that took weeks or even months to build gets blown, but they’ll get over it. If they don’t, they probably aren’t in the right line of work.

We want our tools to always work, but enterprise environments are nothing if not complicated, and the same tool might not work as effectively in one customer environment when compared to another. Chase a broken tool to root cause and discover a new variable to work into future test cycles.

## New Challenges
The focus should always be on creating new challenges for the blue team so they can stretch and learn. So my first advice is always to determine if the campaign can be salvaged as is. Maybe the mistake will be limited in damage to the red team current playbook. That’s OK. Not perfect, but playbooks can be revised. As new information and feedback comes available there is no shame in revising the playbook due to new developments of extenuating circumstances. We are professionals and should act as such!

If it’s only attribution that is blown (e.g. linking a red team operator to a phishing campaign or C&C domain), then simply roll with the original plan, especially if there are valuable training objectives that will be met from the campaign! Since it takes a long time to get from detection to attribution anyway, and the former is more important than the latter.
* **In 2017 the average time for detection and start of remediation was 294 days!!!**
* In 2018 (at least our ongoing customers) we have seen improvements and this figure drop to 86 days!

Or maybe you just need hit the reset button and try again — incinerate the whole pretext to the ground, create a new one, age your domains, etc. The problem with that is it takes time and time is money. We generally think strategically “Prepare for the worst, hope for the best!” and have a generic backup domain, campaign and C2 infrastructure just incase such a problem arises. Its better and more cost efficient to prepare a back-up plan in advance, than struggling to patch something together last minute, that will effect the quality and outcomes of such an important assessment.

If a key tool failed to perform in the live action of the campaign, spend some time to locate the root cause. Then simply move on. Own the mistake — don’t say “the stupid tool didn’t work correctly!” Instead say: “We identified the fault, we can add X functionality to improve its use the reliability next time around!” An understanding Blue Team should recognise the damage the tool could have made and still learn something from the experience.

## Purple Team Mistakes
If the red teamer is afraid of looking like an amateur to the blue team — try to remember just how hard and difficult the Blue-Team job can be. Odds are massively stacked in favour of the adversary and the way you feel is probably how they feel every day. Stay humble, laugh it off, and don’t let your preconceived notions come into play. It is probably good for blue to see red struggling on occasion. In fact, this is a great opportunity for team building during the debrief. If it helps, tell your trusted agents in advance that “the red team” (no need to single anyone out) made a mistake that will result in premature attribution. Then you can all laugh it off later. This may put blue in a better position to laugh off their future mistakes, which have much more dire consequences than Red-Team anyway.

## Fail Fast
The tech industry loves this phrase: “fail fast.” It’s a buzz word, but it fits well with Red Teams, too. If a mistake was made, so be it. End the campaign early. Collect notes and debrief. Discuss how the mistakes were made. Learn from it, but don’t dwell on the mistake. Just make a note and carry on. Pick a new target, get a new campaign started, and put what you learned into practice. The more cycles, the more that can be learned on both sides. Plus, the more cycles behind you, the farther into the past that mistake will travel. Failing fast helps you put the mistake behind you.

## High Performing Teams
One of my favourite takeaways from the expensive Google study on high performing teams is that a team member must feel safe to make mistakes. If a team member cannot overcome a mistake quickly, laugh it off, and learn from it, without fear of persecution from other team members, then the team will break down and eventually become unproductive. The same is true for Red Teams, and even more so for Purple Teams.

If a red teamer is afraid to try things to make mistakes, the team will grow stagnant, repeat the same TTPs (Threats, Technologies, & Processes) all the time, and become ineffective. Creativity is the human adversaries best asset. Along with creativity is the ability to test a hypothesis, which will fail some percentage of the time. I joke with some of my team that I am going to put “Number of mistakes” on their annual performance objectives — like Einstein once said “a true fool, is someone who keeps making the same identical mistake, expecting a different outcome each time.” Whereas, I prefer my team to own a mistake, and fix it — I don’t care if they failed in X many ways, I want to see how they tackled and overcame the problem, and what they learnt during this process!

> “Nobody wakes up in the morning and says, ‘Today will be the day I’ll screw up my work.’”

To the team members who didn’t make the mistake:see the positive intent in your teammate. Have empathy. Your teammate may have been rushed, may have been temporarily clumsy, may have been ignorant about how something worked, but odds are they didn’t do it on purpose. Put yourself in their shoes, because if you don’t, life has a funny way of forcing you to eventually make your own mistakes! That may even be bigger? This is a perfect time to employ the Golden Rule: treat them the way you want to be treated. When it is your turn to say “Oops, sorry” you want the forgiveness and support of your team. Maybe tell them about a time you screwed up. But whatever you do, don’t push your teammate to the point where they don’t want to risk making mistakes in the future, because if you do, you’re killing your team one brain cell at a time.

## Conclusion
An honest mistake not addressed correctly with trust within the team can ruin everything you have worked hard to build. Talent does not grow on trees, but is instead grown and nurtured. Recruiting good talent is hard. Retaining talent and keeping them engaged is even harder because that job never ends. Your team is watching how you respond, and you are setting the tone whether you realise it or not!

