---
layout: post
title: Creating a Challenge VM
date: 2020-07-15 12:00:00 +0200
img: challenge.png # Add image post (optional)
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
tags: [InfoSec, Technical]
---

I've always been a fan of virtual machine hacking challenges (Challenge VMs / Boot2Roots) and what you can learn while solving them. However, after completing a bunch of them, I was curious about what went into creating one. This lead me to creating some challenge VMs with friends and since then several people have asked me for tips on how to create their own ones. The small [mindmap](https://www.xmind.net/) shown below provides a high-level overview of what will be covered during this blog post!

<p class="imgMiddle">
<img src="/assets/img/mindmap.png"  style="width: 80%" />
</p>

## Idea

Like most things, it begins with an idea. Initially I base the idea on the operating system that the challenge VM will be hosted on. Linux is a popular choice for several reasons; it’s free and redistributable, you can create a fully operational server without paying a dime, you can keep it small which makes it more download-friendly, and there are so many different kinds to choose from. Obviously you should be careful about installing software that is not redistributable. Always check the license! Microsoft Windows operating systems are not redistributable, however there is a way around that.

Once you have decided on the operating system, it is time to choose concept(s) that you want to learn about, or teach other people about. When I create challenge VMs -- after I have the main idea ironed out -- I set the final goal. The goal either be obtaining root access on the virtual machine, or access various flags hidden throughout the challenge VM as a reward for completing one of the concepts. The challenges that you come up with, will define the difficulty level of the VM. Depending on the difficulty that you're going for, the end goal could be to leverage single exploits or to chain vulnerabilities or misconfigurations together to complete the challenge. In my opinion, the best ones keep the concepts close to what you would find during actual client engagements or vulnerabilities that you would find in the wild. Keep in mind that the more difficult the challenge, the more time you’ll be spending creating it and testing it. 

## How? 

Well coming up with concepts that you want to learn can be easy, translating that into something enjoyable for others can be a challenge. In my experience, this comes down to the art of storytelling. How well can you capture the participants attention? How well do you describe what you want them to accomplish? How can you keep them interested in the challenge?


Images / notes / videos / gamification 

## Organisation

As with work, when creating challenge VMs I find that the one thing that always helps is staying organised. When coming up with ideas I use Mindmaps to ensure that my process flow works, additionally I make sure that I am providing the participants with the following:

* Background - This is the start of storytelling, why are they trying to solve the challenge and an initial insight into the VM. 
* Relevance - Are the concepts still things that I want to learn? Are they relevant in the industry? 
* Key Information - How the participant should start the challenge and what the end goal is. E.g. is it a boo2root? Are there flags? Is it a series of challenge VMs?
* Ending - Something to tie off the story and make sure that the participant is happy with the outcome of the VM.

Once I have those ironed out, I also make use of a CherryTree document in order to detail every step of the VM - Setup, configurations, challenges, flags (if necessary), etc.

Keeping myself accountable: How long do I have to learn the topic? How long do I want to spend creating the VM? As with project management, you don't want scope creep. Don't try to fit everything under the sun into a single challenge VM, rather spread it out into a series of VMs or have several standalone VMs where each one teaches a new concept. 
Keep a gameplan / timesheet: E.g. I want to complete this in a month, in order to do that I need to set aside x amount of time per week to accomplish this. This post isn't about project management (I might do one about that at a later stage) but it is something that does play an important part in the design process. 


## Example

Once you have your challenge all packaged up, you should test it! Make sure your challenges can be solved, and that everything works as it should. You may find some bugs which need to be fixed or unintended ways of solving your VM. In the end, new vulnerabilities are always being uncovered which could make your challenge VM easier to solve in the future than you initially intended, so even though there might be a number of ways to solve your VM, you should be happy with what you are producing.

## Closing Remarks

Hopefully this little guide will be useful to others thinking of creating their own boot2root. It’s a time consuming process, but it’s very rewarding to see others enjoying a good challenge that you created.