---
layout: post
title: Blue Team Series Part 3&#58; SOC Fundamentals and Escalations
date: 2022-06-28 10:00:00 -0400
img: soc.jpg # Add image post (optional)
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
tags: [InfoSec, Technical, Blue_Team_Series]
---

In the previous post, we looked at what a Security Operations Center (SOC) is and the basic responsibilities of a SOC analyst. In this post, I will be discussing some SOC fundamentals, and types of events and escalations that could occur.

## Indicators of Compromise (IOCs)

Indicators of compromise (IOCs) are “pieces of forensic data, such as data found in system log entries or files, that identify potentially malicious activity on a system or network.” Indicators of compromise aid information security and IT professionals in detecting data breaches, malware infections, or other threat activity. By monitoring for indicators of compromise, organizations can detect attacks and act quickly to prevent breaches from occurring or limit damages by stopping attacks in earlier stages.

Indicators of compromise act as breadcrumbs that are used detect malicious activity early in the attack sequence. These unusual activities are the red flags that indicate a potential or in-progress attack that could lead to a data breach or systems compromise. But, IOCs are not always easy to detect; they can be as simple as metadata elements or incredibly complex malicious code and content samples. Analysts often identify various IOCs to look for correlation and piece them together to analyze a potential threat or incident.
 
Now that you have a basic understanding of what an IOC is, can an IoC ever be "None" when completing a ticket within the SOC? No, The ticket was raised for a reason and that particular reason would be the IOC. Even when it is a false positive, Indicators of compromise (IOCs) are artifacts such as file hashes, domain names or IP addresses that indicate intrusion attempts or other malicious behavior. These indicators consist of:
* **Observables** - measurable events or stateful properties; and
* **Indicators** - observables with context, such as time range.

## Difference Between True and False Positives
When closing tickets, we only note down True or False positives, however True and False Negatives have been included in the definitions below for completeness:
 
#### What are True Positives?
A legitimate attack which triggers to produce an alarm:
* You have a brute force alert, and it triggers. You investigate the alert, and find out that somebody was indeed trying to break into one of your systems via brute force methods.
* If a signature was designed to detect a certain type of malware, and an alert is generated when that malware is launched on a system, this would be a true positive.

#### What are False Positives?
False positives are mislabeled security alerts, indicating there is a threat when in actuality, there isn’t. These false/non-malicious alerts (SIEM events) increase noise and can include software bugs, poorly written software, or unrecognized network traffic.
* You investigate another of these brute force alerts, and find out that it was just some user who mistyped their password a bunch of times, not a real attack.
* If a signature was designed to detect a specific type of malware, and an alert is generated for an instance in which that malware was not present, this would be a false positive.

#### What are False Negatives? 
When no alarm is raised when an attack has taken place.
* There actually was someone trying to break into your system, but they did so below the threshold of your brute force attack logic. For example, you set your rule to look for 10 failed login in a minute, and the attacker did only 9. The attack occurred, but your rule failed to detect it.
* If a signature was designed to detect a certain type of malware, and no alert is generated when that malware is launched on a system, this would be a false negative.

#### What are True Negatives?
An event when no attack has taken place and no detection is made.
* No attack occurred and your rule didn't fire.
* If a signature was designed to detect a certain type of malware, and no alert is generated without that malware being launched, then this is a true negative.

While this definition may seem simplistic to you, if the analysts do not agree on how to label IOCs, it can lead to inconsistencies within the SOC which could ultimately negatively impact the team and the reliability and repeataability of their work.


## Escalations

The SOC analyst’s golden rule for incident escalation is that each escalation must be for activity that cannot be handled by that SOC member in an acceptable time frame. Creating an escalation for a set of alerts or events that could have been handled by the analyst can lead to a couple of debilitating issues.

1. The analyst can end up with the reputation for “crying wolf.”
2. The SOC ends up having too many layers of scrutiny for every potential escalation.
3. More senior analysts may end up dealing with incidents that did not require their attention.

A key component of this golden rule is understanding whether the activity was successful or not. This is not to discount the importance of internally sourced failed attacks. These still need investigation and actioned. But, if you were to choose between successful activity and activity that was blocked, always prioritize by investigating the successful activity first.

### Who do I escalate to?
If a system within an organization has been the target of attack, understanding the business context is critically important to every incident escalation. It’s particularly critical to understand business context from the onset of investigation if the system attacked is a revenue generating component of the organization. This will drive the urgency applied to the investigation as well as the priority assigned to the escalation.
 
Tier 1 SOC Analysts need to review their cases and once they've verified that these events require further investigation, they’ll escalate the issue to a Tier 2 Security Analyst. The Tier 2 Security Analyst can then escalate to a Tier 3 Security Analyst before it ever makes it to an IR/Threat Hunting Team member. Just because the incident went through these steps in the process, does not mean that the analyst that this was escalated to needs to handle the case. If they believe that it requires further escalation, the analyst should escalate as quickly as possible to ensure that the event is dealt with in a timely manner. 
 
In the event of a CSIRT, the Tier 3 Analyst and SOC manager will need to ensure that all relevant information has been captured before raising the issue. This will require input from all parties involved and due to the severity of the issue, no information can be omitted at this stage.


### What Should I Handover?
When escalating an incident, it is essential to ensure all information discovered during the analysis phase is captured in the case you’re escalating. Providing a complete picture of what, when and how the incident happened will quickly bring the team up to speed and will provide clarity about why a particular case was escalated.
 
When handing a case over to another analyst, based on escalation or just case handover, you should have the following information available for them:
**Notes** - The notes should follow a similar structure to basic threat hunting:
* Create hypothesis
* Timeline (Refer to What is a Timeline below)
* What steps you would take next?
* Any relevant information

**Relevant Information** (where possible):
* Affected host, source and destination hostnames and IP addresses
* Source and destination ports
* Operating systems
* System criticality and role
* URL to relevant resources
* Sandbox information
* What logs have you searched for, etc.
* The types of behavior alerted on

Next, be sure to supplement any incident escalation with all additional sources of information used to form your view of the incident and its importance. If evidence was discovered of a successful attack, as mentioned above, then include samples of that data in your case. When you back up an incident escalation with hard evidence, the incident will receive the attention and urgency it deserves.

Do not forget to include items not found during the investigation. For example, if vulnerability information or information on the source and destination of the attack was not available for one of the systems, mention that in the investigation notes. This lets the next analyst or incident responder know that information has already been searched for, so that they are not duplicating efforts.

### What is a Timeline?
At a high-level, a timeline is a summary of the incident. Writing a great summary for any incident escalation is an art and something that comes with time and practice. SOC security analysts frequently struggle with this. They face the challenge of needing to succinctly write a few paragraphs that include a high volume of extremely detailed information and provides details of why this incident is worthy of Incident Response.

A timeline is a great tool that will help you tackle writing effective summaries. Plotting the events discovered during the analysis onto a timeline will help you clarify thinking about the attack progression. This will help form a hypothesis of what you believe took place during the attack based on the facts accumulated. The timeline will greatly assist in more clearly describing the sequence of events in the incident summary.


## Why is this Useful?
In order to be effective in any role, you need to understand the fundamentals of what you are trying to accomplish. By understanding different IOCs, when to escalate, and how to hand over tickets, the SOC analysts will not only learn what is required from them, but they will also be able to understand what they would require from other analysts if a case was ever handed over to them.

## Coming Up

In the final installment of the Blue Team Series, I will discuss methods for note taking within the SOC and potential SOC projects. If you have any questions or if you would like me to include specific content in the series, please do reach out!