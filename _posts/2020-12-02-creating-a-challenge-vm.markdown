---
title:  Creating a Challenge VM
author: kyhle
date: 2020-12-02 12:00:00 +0200
categories: [InfoSec, Technical]
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
image:
  path: /assets/img/challenge.png
  width: 800
  height: 500

--- 



I have always been a fan of virtual machine hacking challenges (Challenge VMs / Boot2Roots) and what you can learn while solving them. However, after completing a bunch of them, I was curious about what went into creating one so I decided to create some challenge VMs with friends. Since then, several people have asked me for my thought process as well as for tips on how to create their own ones. The simple [mindmap](https://www.xmind.net/) shown below provides a high-level overview of what will be covered during this blog post!

<p class="imgMiddle">
<img src="/assets/img/mindmap.png"  style="width: 100%" />
</p>

## Idea

Like most things, it begins with an idea. Initially I base the idea on the concept(s) that I want to learn or teach other people about. A large portion of the time (for me), this initial idea determines the operating system that the challenge VM will be hosted on. However, if you prefer to start off with an operating system instead, Linux and Windows are often popular choices because you can learn so many different concepts on each OS and they are pretty easy to get your hands on. Just keep in mind that in addition to the OS, the software that you are using may require licensing which can become a hassle if you intend to distribute your challenge VM to the public. However, if you are using it for yourself as a means to learn or if you are using it internally within your organisation for training purposes, then there shouldn't be many issues as most software (and Windows versions) have trial periods.

When I create challenge VMs -- after I have the main idea ironed out -- I set the final goal. The goal could be obtaining root access on the virtual machine, obtaining various flags hidden throughout the challenge VM as a reward for completing one of the concepts, or anything else that you want the goal to be. Remember, the challenges that you come up with will define the difficulty level of the VM. Depending on the difficulty that you're going for, the end goal could be to leverage individual exploits or to chain vulnerabilities and/or misconfigurations together to complete the challenge. 

## How? 

In my opinion, the best challenge VMs keep the concepts close to what you would find during actual client engagements or vulnerabilities that you would find in the wild. Keep in mind that the more difficult the challenge, the more time you’ll be spending creating it and testing it. While coming up with concepts that you want to learn can be easy, translating that into something enjoyable for others can be a challenge. In my experience, this comes down to the art of storytelling -- how well can you capture the participants attention? How well can you describe what you want them to accomplish? Can you keep them interested in the challenge?

As mentioned above, all of these questions can be answered through storytelling. The story that you are describing to the user can include clues, links, breadcrumbs, you name it, but there needs to be something for the hacker to chase. You can use all of your creative power during this phase -- if you can think it then there is more than likely a way to have that vision come to fruition. You can include; images, notes, videos, gamification, etc. The sky really is the limit. 

### Organisation

As with work, when creating challenge VMs I find that the one thing that always helps me focus on the task at hand is staying organised. While thinking about new ideas, I use mindmaps to ensure that my process flow works, additionally I make sure that I am providing the participants with the following:

* **Background** -- This is the start of the story, why they are trying to solve the challenge and some initial insight into the VM. 
* **Relevance** -- Are the concepts still things that I want to learn? Are they relevant in the industry? 
* **Key Information** -- How the participant should start the challenge and what the end goal is. E.g. is it a boo2root? Are there flags? Is it a series of challenge VMs?
* **Ending** -- Something to tie the story together.

Once I have those ideas ironed out, I also make use of a CherryTree document in order to detail every step of the VM: 
* **Setup** -- This includes the OS that I have decided on as well as any users, scripts, applications, etc. and the mindmaps from before.
* **Configurations** -- How I introduced each concept and how did went about misconfiguring each step in the exploit chain.
* **Challenges** -- A detailed description of how to exploit each piece of the puzzle, and how I created each link the exploit chain. 
* **Flags (Optional)** -- If this is going to form a part of the VM, it is a good idea to keep track of these in a notes file so that you don't forget about them.

### Keeping Myself Accountable

While creating a challenge VM it is important to keep yourself accountable. As someone who has a number of projects up in the air at any given point, I find it essential to put a gameplan together before tackling a new project. This gameplan will generally include some form of time management and deadline, e.g:
* How long do I want to spend creating this VM? E.g. a few weeks to a few months.
* I want to learn this new concept -- how long do I have to learn it to include it within this VM?
* I want to complete this in a month -- how much time do I need to set aside per week to accomplish this?

The list goes on and on and it should be tailored to your specific requirements. As with project management, you do not want scope creep so do not try to fit everything under the sun into a single challenge VM, rather spread it out into a series of VMs or have several standalone VMs where each one teaches a new concept. This post isn't about project management (I might do one about that at a later stage) but it is something that does play an important part in the design process. 

### Testing

Once you have completed your challenge VM, you are happy with the flow of the challenge, and you are happy that it is describing the story that you wanted to tell, you should test it! Testing is an important part of creating anything since it will ensure that your challenges can be solved and that everything works as intended. You may find some bugs which need to be fixed or unintended ways of solving your VM along the way. In the end, new vulnerabilities are always being uncovered which could make your challenge VM easier to solve in the future than you initially intended, so even though there might be a number of ways to solve your VM, you should be happy with what you are producing before releasing it to your selected audience.

That's it, that is the process I generally follow. I know that most people prefer to work from examples rather than ideas, so I have included an example below. If you are reading this post because you are creating your own challenge VM, try to remember why you are creating it, and most importantly -- HAVE FUN!

## Example of my Current Challenge VM

As a Proof of Concept, I decided to include my planning and thought process for the current Challenge VM that I'm working on in order to detail the development process that I went through. 

### Background
As it is nearing Christmas time, I decided that I wanted to have a challenge VM with a Christmas theme, this will be included within the story components as well as with the potential web application that will accompany the VM. I also decided that since it is Christmas themed, it would be fun to have each of the user accounts that the hacker need to compromise, being names Chris. 

### Relevance
Now that I have a bit of a background into the high-level themes for the VM, I needed to decide on the concepts I wanted to learn. I'm currently interested in learning about Shim Databases in Windows as well as the SePrivileges. Based on this, I am most likely going to make use of a Windows OS and I will include those concepts within the VM but how does that look? How will the user get a foothold? How many loops do you want them to jump through? 

#### I start off with the ports I initially want the user to access
During this planning phase, I decide on how I want the users to interact with the machine. For this Challenge VM I went with the following:

* Open ports: 21, 80, and 5985 
    * 21 -- Anonymous FTP with some basic information (Maybe hints to a hidden directory and username as the user signs off an email).
    * 80 -- Christmas themed IIS web application with either an exploitable vulnerability or a way to retrieve a users password.
    * 5985 -- WinRM port for remote access.

#### Relevant Users
The next part of this phase is determining which accounts I want the user to be able to interact with and during which portion of the challenge they should be accessible to the user. For this part I decided on the following:

* Unauthenticated &rarr;  Anonymous FTP and www-data from web application exploit or credentials found within the application for WinRM authentication &rarr; Low Privileged user.
* Authenticated &rarr; Chris 1 through CMDKey &rarr; Chris 2 through Shim DB exploit &rarr; Administrator through sePrivilege Escalation.

### Key Information

Cool, the bigger picture is starting to come together! Now why do I want FTP open? This will mostly be to add to the story and it is a nice way to provide the hacker with some useful information. Maybe this information will be in the form of notes left behind from Chris 1 explaining CMDKey or something about Shim databases. Why do I want a web application? The Christmas themed web application will also include some information about the additional concepts not covered within the notes. For example, one of the web pages may include hints about SePrivileges or possibly a page with a information about Evil-WinRM.

In order to guide the user to unknown and often overlooked concepts, there need to be hints / pointers to keep them invested in solving the challenges. The worst thing that happens when joining the security scene is having waaaaay to many things to look into without any guidance or hints on how to proceed. Obviously if the objective is to make a really difficult challenge VM then the approach will be different, but I really enjoy guiding people and mentoring them, so the storytelling route appeals to me. 

Within each user's home directory, I will also include a Christmas themed note, which will provide them with some information about the next part of the challenge, as well as some story based information following on from my Christmas theme. So how does this look from a high-level? 

<p class="imgMiddle">
<img src="/assets/img/Overview.png"  style="width: 60%" />
</p>

### Ending

Since this will be a boot2root VM and not a series of Christmas themed VMs, the end goal will be to gain Administrator/System level privileges. However, since it's almost Christmas time and Kringlecon will no doubt be in full swing, the final flag I will include links to the [TryHackMe advent](https://tryhackme.com/christmas) challenges, [SANS Kringlecon](https://holidayhackchallenge.com/2020/) challenges and the [SANS Twitter](https://twitter.com/KringleCon) page so that anyone that had fun hacking their way through this will have additional Christmas themed VMs and challenges to work through.

## Closing Remarks

That's it, that's the way that I worked through creating a challenge VM and the story to go with it. Everybody is different, and just because this method works for me, does not mean that it will work for you. That being said, I do think that the concepts behind organisation and having a goal in mind will be useful, regardless of the method you decide to use when creating your own VM. 

Hopefully this little guide will be useful to others thinking of creating their own challenges. It is a time consuming process and it is very easy to try and add too much to your VM if you do not have a game plan at the start, but in the end it is very rewarding to see others enjoying a challenge that you created and learning a ton of new things in the process.

Happy Holidays & Happy Hacking, Everyone!!