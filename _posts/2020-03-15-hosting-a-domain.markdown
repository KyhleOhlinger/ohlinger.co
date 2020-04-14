---
layout: post
title: Hosting a Domain
date: 2020-03-15 00:00:00 +0200
img: hosting-a-domain.jpg # Add image post (optional)
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tags: [InfoSec]
---

Since setting up this blog, I wanted to write a short post on what I did to get everything up and running. This post isn't going to be super detailed, but I will reference other blog posts which cover the various aspects and questions that I had when buying my domain. 

Before jumping into the details, why buy a domain? Why go through the effort? Well if you're like me, it's to learn something new, with the added benefit of owning something and being able to create whatever you want. For me it also serves as an outlet. I enjoy researching various topics and having a domain with a website allows me to share what I've discovered, or what I've learnt, or even just what I've found interesting at that point in time.

### But how do we begin? 

You need to start off by buying a domain name for yourself or your business. From there, you can create a website, email address, etc. which will allow people to find you online. When I bought my domain, I wanted something associated with my surname which is why I bought [Ohlinger.co](https://ohlinger.co). Buying a domain might seem really complicated, but the [How to Buy a Domain Name](https://www.godaddy.com/garage/how-to-buy-a-domain-name/) blog post covers the main concepts in a step-by-step process which is easy to follow. I used [GoDaddy](https://godaddy.com/) when I bought my personal domain, however I've used [Uniregistry](https://uniregistry.com/) for work before. I honestly prefer the management interface associated with Uniregistry but GoDaddy's prices are better and I believe that they are in the process of acquiring Uniregistry.


### What's next?

After purchasing a domain, you can now use it for a couple of different things, currently I'm using it for email and hosting a website. The first step for me - after purchasing the domain - was getting an email account set up. For this I'm currently using [GSuite](https://gsuite.google.com/). The [Following guide](https://support.google.com/a/answer/33353?hl=en&ref_topic=4445319) provides the steps I used to set up my MX records in order to link the new domain to an email address. 

Obviously, you are able to host your own mail server. But for simplicity I decided to start off with it being hosted using GSuite so that I didn't have to fight with mail verification settings and the like, as described in my blog post on [Mail Verification](https://ohlinger.co/infrastructure/security/2020/03/27/mail-verification.html).


### Hosting a website

The last step that I am going to speak about is hosting your website. There are a bunch of free options that you can use, including [AWS Free Tier](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc). But since I've set up a bunch of VMs before, I decided that I wanted to try something different. A friend of mine mentioned [Github Pages](https://pages.github.com/) (also free) which sounded interesting and I wanted to challenge myself to learn something new.  I found a post on [Using custom domain for GitHub pages](https://medium.com/@hossainkhan/using-custom-domain-for-github-pages-86b303d3918a) when I was initially looking to link my domain, and it has really good information on how to go about hosting your website with Github.

### Hosting your own Infrastructure
There are a bunch of other security considerations to keep in mind, from infrastructure to web application security to mobile security if you go that far. If this is something that you are looking to do, build everything with Security in mind. Security can't be an after thought, you need to dev it in from the start and make sure you have the basics down. I'm going to list a few important items below, but it's is by no means comprehensive.

* Restrict traffic to administrative ports to IPs that you own / use.
* Only open ports that are required, e.g. 22 and 80/443.
* Use SSH Key-Based Authentication instead of password authentication
* Use MFA!
* If you are looking to host your own mail server, keep the information I provided in my Mail Verification post in mind.
* If you are planning on creating a website that requires authentication, use TLS/SSL.

### In Summary

This was a very high-level take on the approach I followed when buying and setting up my personal website. If you have the time, I would highly recommend hosting a VM using AWS, Azure, or on your own infrastructure and playing around with it. I think you can definitely learn a lot from messing around with new technologies and the best way to do that is to set up everything from scratch.

