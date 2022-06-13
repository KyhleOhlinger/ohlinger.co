---
title: Automation Series Part 5&#58; SocBot Release
author: kyhle
date: 2021-10-26 12:00:00 +0200
categories: [InfoSec, Technical, Automation_Series]
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
image:
  path: /assets/img/mystery.jpg
  width: 800
  height: 500

--- 


Information Security (InfoSec) is generally "red team" focused and one of the main reasons is that in the red team space, people tend to share ideas. There is a ton of collaboration, people open source tooling, and communities get involved in order to solve a challenge. On the blue team side, most people tend to stick to themselves. Very few Open Source projects are created, and even fewer are actually useful. If you want to have something for security defence, it generally costs a lot of money and you are then tied to a vendor who attempts to sell you more products over the years.

The goal of SocBot is to show that Open Source automation can be used within the blue team space in order to improve an organisations security posture at no cost other than time and hardware requirements. This project makes use of TheHive and a Telegram bot in order to show you that it is possible to automate processes within the Security Operations Centre (SOC) within smaller organisations who are not able to afford the most high-end Case Management platforms.

The [SocBot Github project](https://github.com/KyhleOhlinger/SocBot) that I created contains a starting point -- an idea -- that can be expanded upon by organsations who are looking to improve their security posture. This blog post will cover the current implementation and ideas to make it actually useful within an organisation. Information on how to configure the project is detailed within the Github Project.

## Why is this Useful?
Organisations with SOC's or Incident Response (IR) personnel will receive alerts throughout the day. There could be several alerts when you are not at your computer and these alerts may be urgent. The main idea behind SocBot is to automate some of the tedious work through the use of Telegram API calls. 

This won't replace the current workflow, but it would make it easier on the employee since they will be able to perform initial checks and determine if it really is serious and they need to jump onto their PC in order to remediate the issue, or if it was just a low fidelity alert (false positive) that can wait until they are back at work. 

## Current Implementation
Due to a ton of other things that have been going on in my life, this project came to a bit of a standstill since the last blogpost and I realised that if I don't release it now, it will just sit and gather dust. Therefore, this is nowhere near what this project could be, but I hope that many of you will take the time to add to the project and improve on the areas that I have not been able to dedicate time to. The current implementation provides an automated Case Management platform which has been integrated with Telegram in order to provide smaller organisations with a starting point which they can use to improve their SOC's capabilities and simultaneously reduce their workload. 

SocBot makes use of the following open source stack:
* ElasticSearch - Part of ELKStack
* Telegram
* TheHive
* Cortex                      

### Current Functionality
This is the culmination of the Automation Series and as such, the functionality is inline with the previous blog posts. After installation, you will be able to create a Telegram bot and link it to the Cortex Analyzers that you want to make use of. The current implementation of the Telegram bot is not context aware, and you can interact with it using the following commands:
* `/start`
* `/help`
* `/vt <hash> / <ip or domain>`
* `/th`

Using TheHive, you will be able to create new cases with the provided Indicators of Compromise (IoC). I'm not going to dive into the functionality behind TheHive -- if you are interested in it as case management software, I would recommend looking at their website which contains a ton of information on how it is used: <https://thehive-project.org>.

## Ideas to make it Useful
As described above, this project is currently very bare bones, but it can be expanded upon with relative ease in order to be useful for an organisation. 

### Encryption and Principal of Least Privilege
If you are going to be making use of this within your organisation, I highly recommend that you make use of an encrypted channel rather than the current cleartext channels. For both TheHive and Cortex, this can be accomplished through the use of reverse proxies. Additionally, I would recommend creating users and accounts while keeping the principal of least privilege in mind.

### Separation of Current Functionality
The Telegram Bot currently creates new cases within TheHive and from there, a user is able to insert new Indicators of Compromise (IoCs) which will perform reputation checks. It may be useful to include more calls within the Python script to perform individual requests before creating a new case. This way, the analyst would be able to perform basic checks on the IoC and create a new case if necessary.

### Improving the Telegram Bot
The Telegram Bot's functionality at this point is rudimentary. There are a ton of improvements that can (and should) be made if you are planning on using this within you organisation. The code includes the base templates for; new functions, keyboard creation, and user prompts. The first major change that I would suggest is to include various prompts whereby the user is asked for different information when performing case management. TheHive has a lot of functionality and I believe that there is a lot more that you can do with the bot.

### File Upload Functionality
Currently, the implementation does not include file upload and I believe that if a SOC is going to make use of a project like this, file upload functionality would be essential. A possible workflow could follow this process:

Upload file to Telegram Bot &rarr; Store in DB (MongoDB, etc) and generate hash.

Once this has been implemented, it would be useful to have integration with a Sandbox environment, such as [Cuckoo](https://cuckoosandbox.org/) and a disassembler such as [Radare](https://rada.re/n/). In this example, the process flow could look as follows:

Upload file to Telegram Bot
1. Store in DB and generate hash
2. Disassemble file with Radare (e.g. `/radare <hash>`)
3. Call cuckoo with Telegram bot (e.g. `/cuckoo <hash>`)

### Integration with Internal Tooling
If your organisation has the ability to block hosts or IP addresses from a central location, it would be beneficial to include API calls within the Telegram Bot to be able to add the file hashes to a Allow or Block list, depending on the severity of the information retrieved from SocBot. Depending on the systems in use, API calls could be made to block IP addresses and domains, or even isolate machines if deemed to be a significant enough. An example process flow is provided below:

Retrieve File / Hash &rarr; Send to SocBot:
1. Check on different Sources
    * VirusTotal
    * Sandbox
    * etc.
2. Depending on Severity
    * Allow / Block hash
    * Allow / Block IP or domain
    * Isolate Machine

Additionally, this could be refined within an organisation to include:
* Integration with AV and SIEM                        
* Integration with ticketing systems such as Compass / Jira

## Conclusion
If you are looking for open source tooling to improve the efficiency of your SOC, automation is key. This project is a very simple demonstration of the power that is possible if your organisation is willing to put in some time and effort into R&D. I believe that with the number of alerts that a company sees on any given day, processes that make the work less tedious and allow the security team to reduce noise are essential in maintaining a confident and invigorated workforce.

If you have any questions regarding the implementation of the Github project, please feel free to reach out to me!