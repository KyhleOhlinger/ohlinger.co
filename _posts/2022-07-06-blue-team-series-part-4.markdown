---
layout: post
title: Blue Team Series Part 4&#58; Note Taking and SOC Projects
date: 2022-07-06 10:00:00 -0400
img: soc.jpg # Add image post (optional)
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
tags: [InfoSec, Technical, Blue_Team_Series]
---

In the previous post, we looked at SOC meetings and escalations. In this post, I will be wrapping up the series with my thoughts on note taking and a review of the series. 

## Note Taking

The purpose of a SOC is to monitor and prevent malicious behaviour within an organisations environment. Additionally, it provides direct support to the cyber incident response process. In order to provide effective support during incidents, a SOC analyst needs to efficiently preserve case information while working through a ticket. 

To accomplish this, the analyst needs to be able to capture relevant data as they work through the process instead of attempting to reconstruct it after the fact.


### Note Taking Template:
There is a trade off between taking notes effectively and adding time to investigations. The purpose of this template is to organise topics that would be encountered during every investigation into a concise format which could help with future investigations and threat analytics. The fields can be added to a ticket's comment section once the investigation has been completed.
* IoCs: What were the IoCs for this ticket?
* True / False Positive: After investigation, was this a true or a false positive?
* List of Actions Taken: What process was followed?
* Observations by the Analyst: Theories, notes, etc. Any notes that the analyst took during the investigation. This can include screenshots and general comments as the analyst is working through the ticket.


## SOC Projects
SOC projects should be incorporated in every SOC. Not only will this allow the SOC to focus on additional work, but it will also reduce the likelihood of "alert fatigue" that affects the majority of SOC analysts. 

SOC projects should encourage collaboration within the team and allow each analyst to either work together or individually to upskill themselves, improve the SOC, and provide a creative outlet that they might not get from assessing tickets throughout the day. 

While the types of projects will differ between teams, I have provided a few ideas below that the SOC analysts can take part in, in addition to their role of supporting the business: 
 
* Creating a centralised knowledge repository
* Adapting to a new ticketing model
* Education around SecOps
* Developing Internal training material
* Threat Hunting training

### Creating a centralised knowledge repository

This project is ongoing and the Wiki forms the basis of it. It will include documents, training material, processes, useful information, and anything else that the SOC deems useful. 

### Adapting to a new ticketing model

Currently the note taking and ticket completion aspect of alerting is not defined. This project aims to formalise the process and introduce consistency and repeatability to the way that alerts are handled by the SOC.

### Education around SecOps

This project will take the team through the type of information that can be uploaded or pasted onto public infrastructure like CyberChef. Additionally, if there are publicly available tools that do not meet the requirements set out by FNB, then local instances of those tools, such as CyberChef, will be deployed. 

### Developing Internal training material

There is an abundance of training material available within the field of Information Security, however unless it is condensed to a format that makes sense for an organisation, the information can seem overwhelming. The intent of this project is to create training material for new SOC members to ensure that they understand the internal tooling used by FNB and are able to be onboarded quicker into the team. An example of the training material that can be created is listed below:
 
1. Note Taking Process
2. Basic Threat Hunting	
3. How to use internal tooling	
 
### Threat Hunting training

A virtual server project with the requirements for tooling set out by the SOC team. The architecture will need to be planned out and the device would need to be isolated. Zaid will be able to spin up the instance once the planning has been finalised.


## Conclusion

One of the biggest differences that I have seen between blue and red teams is knowledge sharing. With red team (offensive) content, there  is a massive community, a ton of research is published annually, groups of people work together, and a large portion of scripts and tooling is open sourced. On the other hand, blue team members seem to want to stick to themselves, they want to be in control and horde knowledge in order to make their positions seem more secure, which is often in direct conflict with the goal of a blue team. Additionally, community projects don't really exist and the tooling that is useful is exceptionally expensive. 

This disparity is one of the main reasons (in my opinion) that there is a massive gap in employee retention within the cyber security field, and a almost cult like following for red teamers. I personally believe the only way to have employees want to stay within a blue team is to make them feel appreciated and to show them that they are in fact making a difference, while also giving them the space to learn.

As always, if you have any questions or if you would like me to include specific content in the series, please do reach out!