---
title: Blue Team Series Part 2&#58; SOC Onboarding and Training
author: kyhle
date: 2022-06-08 10:00:00 -0400
categories: [InfoSec, Technical, Blue_Team_Series]
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
image:
  path: /assets/img/soc.jpg
  width: 800
  height: 500
---

In the previous post, we looked at what a Security Operations Center (SOC) is and the basic responsibilities of a SOC analyst. I did mention that I would discuss the meetings that I think help a SOC function as a team, types of events and escalations, as well as potential methods for onboarding and training, however, there was too much to uncover and I did not want any parts to be skipped over. That being said, during this post I will be discussing potential methods for onboarding and training and the rest will be covered in future blog posts. 

## SOC Onboarding
When new hires start, there should (at least) be a rough schedule for them to follow to get up to speed with everything related to the business and their specific role within the team. When creating onboarding training, the team lead / SOC manager should determine what the ideal training schedule looks like and then backfill who handles each aspect of the onboarding. It will be different for hires in different departments, and at differing levels of responsibility, but the goal is to set a framework that you can reuse, bringing in the right people as necessary.

For the SOC, a new hire will be required to complete their onboarding training and basic training, and example of which is outlined below. This training is self paced and it relies on the individual to show initiative when integrating with the team.

### Basic Training:
Within the realm of the SOC, the following training material should be provided to new hires:
* **Internal Tools** - This contains information regarding the tooling that the SOC makes use of daily. It includes information on not only the EDR and logging solutions, but all MSSPs, etc.
* **Knowledge Sharing Review** - The new analyst should review the processes and procedures outlined within the team's central knowledge base. If a shared knowledge base does not exist, the creation thereof should be priority number one for any SOC manager.
* **Use Cases** - This contains information detailing each use case that is alerted on within the SOC. It includes a description of each use case as well as how the analyst is expected to handle the case at a high level.	
* **Playbooks** - This contains information related to the various playbooks and processes that the SOC analysts follow. It includes processes for Phishing, Malware, Threat Hunting, and anything else that is required from the analysts.
* **Threat Hunting** - This contains useful information about what Threat Hunting is and some basic scenarios that Analysts can look at when attempting to perform Threat Hunts within the environment.	

### Skills Acquisition Through Shadowing
Shadowing provides a safe environment for new joiners to learn from more experienced SOC analysts. The shadowing phase within SOC onboarding will ensure that the new analysts are familiar with the processes that they have learned about above, and it can be used as an opportunity for them to ask any questions and become comfortable with the job requirements before expecting them to handle their own incidents. The total time for onboarding should not exceed a week after access to all systems has been granted.

## Additional Training:
Once the new joiners are comfortable with the theoretical knowledge provided during onboarding, additional training for SOC analysts could fall under some or all of the following categories:
* **Product Skills** - Vendor Specific Training
* **Vendor Neutral** - Job role, tasking, basic concepts
* **On The Job** - Internal training with shadowing
* **SOC Exercises** - Monthly training exercises lead by the SOC manager or Threat Hunters
* **TableTop Simulations** - Quarterly events that the Threat Hunters run with different Business Units (BUs)
* **Cyber training** - internal labs, conferences, self-paced learning, etc.
 
### Internal Training:
From an internal training perspective, there should be several projects underway within the SOC including a project focusing on the creation of internal training material for new joiners. This will include vendor specific and vendor neutral training such as [Cyber Range](https://www.cyberbit.com/platform/cyber-range/). This training material should be easily accessible to all analysts and should be referenced within the onboarding process and the centralised knowledge base.
 
Once this training material has been completed, the SOC analyst will have on the job training whereby they should consult more senior SOC analysts and the Threat Hunters.
 
#### SOC Exercises:
Similar to the Ticket Sampling scenarios, a SOC manager or Threat Hunter should take the SOC through different scenarios on a monthly basis to determine how the team would handle specific situations. This will form part of vendor neutral / on the job training.
 
#### TableTop Simulations:
These are events that the Threat Hunters could run on a quarterly basis with different BUs. In order to assist in training and cross-skilling, a analyst will join in on those sessions on a rotational basis. I.e. every quarter a different analyst will be able to sit in on one of those sessions.
 
### Cyber Training:
Cyber training will refer to self-paced training that each analyst takes to improve their capabilities. This can include conference talks, virtual conferences, building offensive and defensive skills, certifications, etc. 

I am a firm believer that in order to become a good SOC analyst, you need to understand what you are defending against. This includes the offensive side of cyber security. In order to become more familiar with this, [HackTheBox](https://www.hackthebox.eu/) and [TryHackMe](https://tryhackme.com) are some of the best ways to upskill in offensive capabilities and defensive. With new machines and challenges released on a weekly basis, you will learn hundreds of new techniques, tips and tricks.

### Mitre ATT&CK Framework
While not directly referenced during most cyber training courses, blue teams make extensive use of the [Mitre](https://attack.mitre.org/) ATT&CK framework. The framework provides an amazing breakdown of the tactics and techniques which are used by attackers when targeting an organisation. The website makes use of several matrices which cover the TTPs for different OS', cloud platforms, and mobile. It also includes a table which covers the processes used during Reconnaissance and Resource Development. When referencing the framework, it is important to understand the cyber kill chain which follows the phases of a cyber attack: from early reconnaissance to the goal of data exfiltration.  A large number of organsations utilise the Lockhead Martin variation of the cyber kill chain which encompasses the following:
  
* **Reconnaissance:** Intruder selects target, researches it, and attempts to identify vulnerabilities in the target network.
* **Weaponization:** Intruder creates remote access malware weapon, such as a virus or worm, tailored to one or more vulnerabilities.
* **Delivery:** Intruder transmits weapon to target (e.g., via e-mail attachments, websites or USB drives)
* **Exploitation:** Malware weapon's program code triggers, which takes action on target network to exploit vulnerability.
* **Installation:** Malware weapon installs access point (e.g., "backdoor") usable by intruder.
* **Command and Control:** Malware enables intruder to have "hands on the keyboard" persistent access to target network.
* **Actions on Objective:** Intruder takes action to achieve their goals, such as data exfiltration, data destruction, or encryption for ransom.

## SOC Meetings

One of the biggest differences that I have seen between blue and red teams is knowledge sharing. With red team (offensive) content, there  is a massive community, a ton of research is published annually, groups of people work together, and a large portion of scripts and tooling is open sourced. On the other hand, blue team members seem to want to stick to themselves, they want to be in control and horde knowledge in order to make their positions seem more secure, which is often in direct conflict with the goal of a blue team. Additionally, community projects do not really exist and the tooling that is useful is exceptionally expensive. 

This disparity is one of the main reasons (in my opinion) that there is a massive gap in employee retention within the cyber security field, and a almost cult like following for red teamers. I personally believe the only way to have employees want to stay within a blue team is to make them feel appreciated and to show them that they are in fact making a difference, while also giving them the space to learn.

While meetings can be detrimental to the work day, there are certain meetings that I believe are required for teams to function as a unit, as opposed to individual members. I personally recommend daily and weekly meetings, the purpose of which is described below.  

### Daily Meetings:
The SOC has two daily meetings, one at 10:00 and another at 15:00. The purpose of the daily SOC meetings is to cover:
* Key Events
* Status of ongoing incidents
* Major data issues
* Relevant communication topics
* Reoccurring alerts or incidents
 
These meetings will not only allow the other members to see what is happening outside of their work, but it also encourages collaboration within the team. These meetings should be scheduled for roughly 30 min, depending on the size of the team. Additionally, these meetings should aim to provide value to everyone involved. An example of how to do this, includes having SOC talks with the team and sampling tickets during the daily meetings. 

**SOC Talks** - Where possible, the analysts should be able to showcase a technology or technique that they found useful, to the rest of the SOC. This is going to provide an opportunity for knowledge sharing and hopefully it will serve to build up the collaborative spirit.
 
**General Ticket Sampling** - every few days at the 3pm meeting, run through one or two incidents as a team. 
* Post incident review
* Ticket review / quality control
* Questions to be answered per ticket need to be formalised in a centralised knowledge sharing board
 
**Friday Ticket Sampling** - every Thursday the analysts should provide a link to the most difficult or interesting case that they worked on during the week. From there a single ticket will be chosen and worked through with the team on a Friday afternoon during the team chat.

### Weekly Catch-ups:
Every week the SOC members will catch-up with their team lead / SOC manager. The purpose of this meeting will be to ensure that the members have a forum in which they can raise any questions or concerns that they may have. As a follow on from these weekly catchups, I recommend that weekly reviews are conducted by each analyst.

The review should cover:
* What went well?
* What went badly?
* What did you struggle with?
* What can be improved upon to make next week better?
  
This can be done in isolation or used as talking points during the weekly catch-up but the SOC manager should make an effort to get feedback from the team in order to make improvements to the working environment throughout the year. 

## Why is this Useful?

The purpose of the SOC onboarding, Training, and Meetings is to ensure that the new joiners are provided with all the information that they require to succeed within their role. By having meetings and shadowing opportunities within the SOC, it will not only reduce the time required for training, but it will also enforce the idea of a team.

## Coming Up

In the next installment of the Blue Team Series, I will discuss the types of events and escalations that SOCs could face on a daily basis. If you have any questions or if you would like me to include specific content in the series, please do reach out!
 

## Useful Resources:
* [Blue Team Handbook](https://www.amazon.com/Blue-Team-Handbook-condensed-Operations/dp/1726273989): Contains useful information about SOC operations and usefulness of SIEM products. I'm pretty sure that there are copies available online.
* [Mitre SOC Strategies](https://www.mitre.org/sites/default/files/publications/pr-13-1028-mitre-10-strategies-cyber-ops-center.pdf): The Mitre framework on the Top 10 world class strategies for SOC operations. 
* [Creative Choices](https://chrissanders.org/2019/10/creative-choices-paper/): A really good white paper which  attempts to understand how security analysts think during the investigation process.
* [Computer Security Incident Handling](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf) A publication from NIST on incident response plans and how an organisation should handle events.


If you are interested in Threat Hunting, the following resources can help:
* [Threat Hunting Basics](https://medium.com/@jshlbrd/threat-hunting-basics-68fb1980cc9b)
* [Understanding Cyber Threat Intelligence](https://www.bankofengland.co.uk/-/media/boe/files/financial-stability/financial-sector-continuity/understanding-cyber-threat-intelligence-operations.pdf)
* [The Threat Hunters Handbook](https://cyber-edge.com/wp-content/uploads/2016/08/The-Hunters-Handbook.pdf)
* [The Pyramid of Pain](http://detect-respond.blogspot.com/2013/03/the-pyramid-of-pain.html)