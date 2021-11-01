---
layout: post
title: "Review: CyberWarFare Cloud Red Team"
date: 2021-11-01 12:00:00 +0200
img: cyberwarfare.png # Add image post (optional)
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
tags: [InfoSec, Technical]
---

Hi everyone, 

I do not normally perform reviews of courses, but I thought that the Cloud Red Team course by CyberWarFare warranted one. So as a prelude to this course, I did take the [AWS Cloud Practitioner](https://aws.amazon.com/certification/certified-cloud-practitioner/) course offered for free by Amazon, as well as the [AWS Solutions Architect Associate](https://www.udemy.com/course/aws-certified-solutions-architect-associate-hands-on/) course on Udemy, offered by Neal Davis. Both of these courses, in my opinion, covered a massive amount of content for a reasonable price. However, if you have not done an AWS course before, then the introduction lectures from the CyberWarFare course would suffice for a basic understanding of the AWS services covered within the course videos and PDF.

Why did I take this course? After completing the aforementioned AWS courses, I started to look for an Offensive AWS course and came across the [Certified AWS Cloud Red Team](https://www.CyberWarFare.live/certified-aws-cloud-red-team-specialist) by CyberWarFare. The course description was probably the best I had seen for an offensive cloud course, even if the price was a bit steep, so I thought I would give it a shot. Before I dive into the review, I purchased the Gold package which did not include the AWS Cloud Red Team Lab access. However, based on the content and the quality of the exam, I do not believe that the lab would be worth an additional $300.

While there are not a ton of pros for the course, there are way too many cons for me to recommend this to anyone, including:
- The course is expensive 
- Course quality is poor and commands can be found by looking at AWS CLI documentation
- The course does not accurately reflect an actual cloud Red Team engagement
- The exam covers topics and techniques that were never described during the course videos or PDF
- The duration for the Gold CARTS exam is confusing and what is required to pass is unclear

## The Course Outline by CyberWarFare 
> AWS Cloud Red Team Course provides in-depth view of AWS core services, Identification of mis-configurations and stealthily abusing them in an Enterprise AWS Cloud Environment. As cloud shift is real, most of the Fortune 500 enterprises relies on AWS Cloud service provider for scaling their business overseas, with expansion comes huge responsibility of identifying and mitigating wide loopholes to secure cloud infrastructure.

The description of the course sounded great and it also included a PDF manual as well as 20+ hours of video content which would allow students to understand the basic concepts and follow along with the practical aspect of the course. Based on the description, the course is meant to:
-   Align with MITRE ATT&CK Cloud for AWS
-   Enumerate & Design AWS Cloud Attack Surface
-   Learn & Understand core AWS Cloud services Functionalities
-   Pivoting & Lateral Movement using AWS VPC
-   Post-Exploitation by abusing mis-configured AWS Services

Additionally, the course was meant to be set out in such a manner that it covered the various aspects of the cyber kill-chain:
- Initial Access
- Execution
- Persistence
- Credential Access
- Discovery
- Lateral Movement

While I am not going to go into detail about each service covered by the course, the list of services that have course content associated with them are provided below:
- Identity And Access Management (IAM)
- Virtual Private Cloud (VPC)
- EC2 (Elastic Compute Cloud)
- Lambda
- Containers
- Simple Storage Service (S3)
- Relational Database Service ( RDS )
- Elastic Block Store ( EBS )
- Secret Manager
- Single Sign On (SSO)
- AWS Security Services

This all sounded really appealing to me since that breakdown of exploitation was what I was used to from an Active Directory (AD) perspective, and it would have made it really easy to adapt to cloud based persistence and exploitation techniques. 

### The Course in Practice

I personally think that the course started off strong, the content for the Identity and Access Management (IAM) portion was pretty decent and it was inline with what I expected from a "Red Team" course. However, after the IAM section, everything started going downhill. The content became sloppy and I could have arrived at the same conclusion by just looking through the AWS CLI commands to perform enumeration for the remaining services. Additionally, every section followed the assume breach methodology and then basically just performed enumeration. I feel like this is a pretty poor way to run a course like this. 

The video quality was poor and the lack of preparation by the instructors was evident in the videos. If you want a comparison, look at their [YouTube channel](https://www.youtube.com/channel/UC4X1CYxw0fDIpFX5zmC8WNg/videos) which has the first part of the course for free, and compare it to any free video on YouTube for any Cloud security conference. You will quickly discover that the quality of the course and videos is subpar at best. Additionally, the final sections (SSO and AWS Security Services) only contained an introduction into the services and did not contain any useful information for red team operations.

## The Labs
I bought the Gold version of the course which didn't include access to the labs. Based on the course content and the exam, I honestly don't think that the labs would useful, but I could be wrong. Based on the content of the course videos and, I think that the anyone looking to purchase the labs would be better off saving their money and spinning up some [Cloudgoat](https://github.com/RhinoSecurityLabs/cloudgoat) practice labs.

## The Exam
As I mentioned at the start, I didn't do the labs and since the exam was only 2 hours long, I had no idea what to expect:
> Candidates will be judged to pass a 2 hours hands-on examination on an Realistic AWS Enterprise environment to earn AWS Cloud Red Team Diamond Certification.

When I booked for the exam however, I was surprised to see that the response from CyberWarFar Labs was that the exam was actually **24 hours long**:

> The exam is 12 hours of fully hands-on experience, you will have 24 hours to document & submit your findings in PDF format to us at support@CyberWarFare.live

Okay, at this point I got a bit more excited. I didn't think that 2 hours would actually provide anything useful, so I was glad that the exam style had changed. That being said, I still had no clue what to expect, so I started going through the course content again and making some Python scripts. Additionally, I read through a Cloudgoat walkthrough which covered some concepts that I was unclear about.

### Take 1
I started my exam at 12:30 IST (all exams are booked in IST). Without giving anything away, it started with user credentials and I needed to enumerate my way into compromising a service. I had a number of setbacks with the lab due to internet connectivity and the actual session token that was provided timed out. At this point I contacted their support team and they took over 2 hours to reply to me and said that they would cancel my exam and I would need to reschedule:
> We have decided to end your examination here and we have re-scheduled the examination. 

This was extremely demotivating and I was very disappointed that they were unable to just provide new credentials since I had to take time away from work in order to accommodate this exam. After this, I asked some more questions and got confusing replies from their team: 
> For CARTS Gold certification, Exam duration is only 2 hrs. You can reschedule your exam according to your availability. 
> Please note that token is valid for only 2 hrs for  Gold and 12 Hrs for Diamond. 
> Both exam are the same but passing criteria are different. To clear the Diamond exam you need more hands on and long duration.

I found this to be unclear and I continued asking questions and after a while they just stopped replying to me, so I just booked the exam again for the following week.

### Take 2
At this point I had no idea what the exam was actually meant to cover and I still didn't understand how long the examination actually was. I once again started at 12:30 IST. After 2 hours (as per their instruction for the Gold CARTS exam) I stopped and provided them with my report which was not adequate to pass as I did not exfiltrate sensitive information from the AWS environment. The exam was quite a disappointment since the initial path of exploitation relied on enumerating and exploiting a service which was barely mentioned in the course content.

I am still uncertain if this is actually possible with the Gold CARTS version of the course since the exam is only 2 hours long and it is the same exam as the Diamond CARTS exam which is 12 hours long. 

## Conclusion
Would I recommend this course? No, I personally do not believe that the course is worth the money. All in all, the course left me longing for more. I thought that the idea that they had was fantastic and the services covered sounded great, but the implementation thereof left a sour taste in my mouth after spending $299 on a course that didn't add any real value to my career. 

While some of the concepts covered are decent, I do believe that you can gain more knowledge by reading the [Rhino Security Labs](https://rhinosecuritylabs.com/) blog, spinning up the Cloudgoat instance in your own AWS environment, and playing around with the IAM permissions tool that Rhino Labs created while looking at the AWS documentation for interacting with the AWS CLI. 

Additionally, the examination is confusing and there is no clear advice provided at any stage on what you are actually attempting to achieve within the Gold CARTS examination. I was extremely disappointed in this and I will not be taking another CyberWarFare course.