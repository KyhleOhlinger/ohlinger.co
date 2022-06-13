---
title: Blue Team Series Part 1&#58; What is a SOC?
author: kyhle
date: 2022-05-12 10:00:00 -0400
categories: [InfoSec, Technical, Blue_Team_Series]
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
image:
  path: /assets/img/soc.jpg
  width: 800
  height: 500
---

The first blog post in this series is going to be very short as it aims to cover the basics of what a Security Operations Center (SOC) is and the basic responsibilities of a SOC analyst, which will lend itself to the blog posts that follow. 

A SOC can be defined a number of ways depending on who you ask. For my simple definition, a SOC is "a team that protects an organisation against cyber threats". While this is very simplistic, it does dictate what the main purpose of a SOC is, which is to defend an organisation. 

How the SOC does this can differ, but it generally encompasses; monitoring, detecting, preventing, analyzing, and responding to potential threats against the organisation's digital assets. Depending on the size of the organisation, they may have an internal, external or hybrid SOC, and this may or may not be 24/7. For this series, let's assume that the SOC runs on a hybrid model, whereby the focus will be on an internal SOC which works normal business hours (08:00 - 17:00). Additionally, they could receive alerts from the external SOC which monitors the network 24/7.

### The Basic Responsibilities of a SOC

Before setting up an SOC, organisations must develop an overarching cyber security strategy that aligns with their business objectives and challenges. If you haven't read the [Blue Team Handbook](https://www.amazon.ca/Blue-Team-Handbook-Condensed-Operations/dp/1091493898), I would highly recommend it to anyone that is looking to work within a Blue Team or lead it.

In order for the SOC to be effective, they need to understand:
* How their work will align with the organisation's Security Strategy
* The current challenges that the security team faces
* What is viewed as a successful day within the SOC
 
Once an understanding of how the SOC will benefit an organisation has been described to all stakeholders, the SOC needs to have goals which prove that the investment from the business has been a successful one. The main purpose of a SOC is to protect the organisation and support the team to reach their goals and objectives. In order to accomplish this, each analyst has the following responsibilities:
1.	**Perform alert triage** - Analysts should follow a priority model as alerts are raised. 
2.	**Work on alerts** - Collect data, investigate, start the ticket or escalate.
3.	**Dashboard review**- Ensure that the queue is being addressed correctly (We are all in this together, try help out when you can).
4.	**Report operational issues** - helps to have the problems addressed at the root. 
5.	**Optimise operational processes** - each member should be able to help with optimisation where possible or suggest areas of improvement.

It is important to remember that when malicious events occur within the environment or when there are escalations from other business units (BU's), it is the SOC's responsibility to ensure that the organisation is protected. This means that even though work hours (in this scenario) are 08:00 - 17:00, if an important incident comes up, it is the SOC that needs to ensure that the event is not malicious before ending for the day.

## Why is this Useful?

While these definitions may seem basic, I have found that a number of analysts do not understand their purpose within an organisation and they do not feel like they add value. My hope is that this series will provide a simplified overview of the role of a SOC as well as potential methods that can be used to not only help the SOC function more effectively, but also work together as a team in order to accomplish a specific goal. 

## Coming Up

In the next installment of the Blue Team Series, I will discuss meetings that I think help a SOC function as a team, types of events and escalations, as well as potential methods for onboarding and training. If you have any questions or if you would like me to include specific content in the series, please do reach out!
