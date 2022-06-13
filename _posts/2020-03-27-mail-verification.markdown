---
title: Mail Verification
author: kyhle
date: 2020-03-27 12:00:00 +0200
categories: [InfoSec, Technical]
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
image:
  path: /assets/img/mail-verification.jpg
  width: 800
  height: 500
---



So I've been interested in learning a bit more about email and the standards which are pushed with it. This lead me to trying to gain a better understanding in what SPF, DKIM, and DMARC are and how they actually work. This post is an overview of the information provided within the *Useful Resources* section at the bottom of this post. So what do the mail verification settings do? The reason that they exist is to prevent spammers from forging the **From** address on email messages. If spammers use your domain to send spam or junk email, your domain quality can be negatively affected. Users who get the forged emails can mark them as spam or junk, and this can impact valid messages sent from your domain.

There are three technologies that can be used to secure mail and protect against spam & spoofing:
* DomainKey Identified Mail (DKIM)
* Sender Policy Framework (SPF)
* Domain-Based Message Authentication Reporting & Conformance (DMARC)

## Sender Policy Framework (SPF):
SPF allows you to specify which email servers are legitimate servers for your domain, i.e. it enables a domain to publicly state which servers may send emails on its behalf. SPF authenticates a sender’s identity by comparing the sending mail server’s IP address to the list of authorized sending IP addresses published by the sender in the respective DNS record. An important aspect to understand about SPF is that it does not validate against the **From** domain. Instead, SPF looks at the Return-Path value to validate the originating server. The Return-Path is the email address that receiving servers use to notify the sending mail server of delivery problems. The problem with this limitation is that the **From** address is what recipients see in their email clients. Furthermore, even if a message fails SPF, there's no guarantee it won't be delivered. That final decision about delivery is up to the receiving ISP.

### So what does an SPF record look like?
* *Basic Example:* `TXT @ “v=spf1 a include:_spf.f-secure.com ~all”`
* *Actual F-Secure SPF:* `v=spf1 ip4:46.228.134.9 ip4:46.228.136.136 include:a._spf.f-secure.com include:b._spf.f-secure.com include:_spf.salesforce.com include:msgfocus.com include:mktomail.com mx:mail6.centercode.com include:stspg-customer.com ~all`

Other domains included within an SPF, e.g. *include:msgfocus.com*, will include their SPF records as well. How does it work in practice? In short, the receiving server extracts the domains SPF record and checks that the source email is approved to send emails for the specified domain.

### What do the parameters mean?

* v - version
* TXT - DNS zone record type
* @ - Placeholder symbolizing the current domain
* a - If your IP address of the A record is used, it will pass the check
* include - Authorises the hosts specified - can also be CIDR notation
* ~all - Who is allowed to send email

### Syntax Action for the all parameter:


| Name | Mechanisms | Qualifier | Intended Action |
|------|------------|-----------|-----------------|
| Permissive          | + | Pass      | Accept | 
| Less permissive     | ~ | SoftFail  |  Accept but Mark|
| Hard Fail           | - | Fail      | Reject |
| No policy statement | ? | Neutral   | Accept|

  &nbsp;

Even though *-all* flag makes this a more secure solution - if the sender does not match the SPF record, the mail will be rejected, which will probably get someone fired because then any mail server not in a pre-approved list will not be accepted and if anyone or any other department within the organisation spun up a mailbox server, then it wouldn't go through.

Why bother if the ISP has the final decision? The problem is without SPF or with a misconfigured SPF record, an attacker can essentially pretend to be part of your organisation and spoof emails that appear to come from your domain. That being said, an email can pass SPF regardless of whether the **From** address is fake. In order to combat this, DMARC was designed to address these issues. Having an SPF policy provides an additional trust signal to ISPs so you can increase the likelihood that your emails arrive in the inbox. The SPF policy can also help mitigate the backscatter of bounce and error notifications when spammers try to abuse your domain. Ultimately, SPF won't solve all of your delivery problems, but it’s an additional layer that, combined with DKIM and DMARC can improve your delivery rates and prevent abuse.

## DomainKey Identified Mail (DKIM):
The goal of DKIM is to ensure that messages weren’t altered in transit between the sending and recipient servers. It uses public-key cryptography to sign email with a private key as it leaves a sending server. Recipient servers can then use a public key published to a domain’s DNS to verify the source of the message, and that the body of the message hasn’t changed during transit. Once the hash made with the private key is verified with the public key by the recipient server, the message passes DKIM and is considered authentic.

Why use DKIM? It's easy to impersonate a trusted sender over SMTP, this can lead to spam for end users which appeared to come from legitimate sources. DKIM aims to make it more difficult to spoof email from domains. DKIM is an optional security protocol, which means that even if the mail is signed, the receiving server doesn't need to have DKIM enabled in order to receive and read the mail. Unlike PGP, DKIM will make sure your message hasn’t been altered, but it doesn’t encrypt the contents of your message. DKIM allows you to attach a DomainKey signature to your outgoing mail. The receiving server then verifies the validity of the key and either accepts or rejects the mail based on the outcome. DKIM protects the **From** address by cryptographically signing messages to verify the author. Usually, DKIM signatures are not visible to end-users, the validation is done on a server level.

### An example DKIM-Signature looks like this:

* DKIM-Signature: `v=1; a=rsa-sha256; c=relaxed/relaxed; d=f-secure.com; h=from : to : subject : date : message-id : references : in-reply-to : content-type : mime-version; s=msg2048; bh=pNY3oJQ1zJQXJI4H13CzpTM1KgTe5eqK6ES0DXoPT+o=; b=PJNYB6C5abyvBIxWGVj3/EOMqJsHxd6vLf0hD24iRKlPBQy4pNrPxaQ3godyVAI1TAd0 xuWVZrggtwZLLKAtAUbrW5HpVzh6Ef3qQqmYiVWPnVPK7HAWrKvdJU224dqav3+dXbVG TQWi668kmCn35y4HNaHMdMzEHzkm9cYRvDJo1Kkqy7FjQBYVB4aBOP7Hm3SdQflSXdEx lGQO2WNIdTnrFjU62WwjmCQFEWqexEUv73+XQoaVPMvIYIdyNSEf51FOGFKCEdNIC3LB iH42UdO3kB5EcsErLQoTvDFCZBcLMPmmN3fdiNMHUKuI98jUXsGOWdG8Igc/CmeUK/R6 tg==`

### What do the parameters mean?
* DKIM-Signature - Header registered for DKIM signed messages
* v - Version of DKIM being used by the sending server
* a - Algorithm used to generate the hash for the private/public key pair. (only rsa-sha1 and rsa-sha256 are officially supported)
* c - Regulates whitespace and text wrapping changes in a message. (Simple - No changes allowed, Relaxed - Allows common changes to whitespace and header line wrapping)
* s - Selector used for the public DKIM key for verification.
* d - Email domain that signed the message (If you're setting DKIM up, the domain name should be used here to improve domain reputation)
* i - Usually in the email address (Identity of the signer)
* bh - Value of the body hash
* b - cryptographic

The private key must be kept secret. If a malevolent user ever gets their hands on your secret key they will be able to forge your DKIM signatures.

## Domain-Based Message Authentication Reporting & Conformance (DMARC):

In essence, DMARC specifies how your domain handles suspicious emails. DMARC is built on top of DKIM and SPF and requires both of them to be set up in order to verify that messages are authentic (DMARC only works if you have both SPF and DKIM configured). Together they are considered best practice in order to prevent email spoofing and make your emails more trustworthy. DMARC ensures that fraudulent emails get blocked before you even see them in your inbox. In addition, DMARC gives you great visibility and reports into who is sending email on behalf of your domain, ensuring only legitimate email is received.

### What does it do? 

DMARC helps email senders and receivers verify incoming messages by authenticating the sender's domain. DMARC also defines the action to take on suspicious incoming messages. To pass the DMARC check:
* Incoming messages must be authenticated by SPF, DKIM, or both.
* The authenticated domain must align with the domain in the message From header address.

When an incoming message doesn't pass the DMARC check, the DMARC policy defines what happens to the message. There are three possible actions:
1. Take no action on the message.
2. Mark the message as spam and deliver to recipient's spam folder.
3. Tell receiving servers to reject the message.

### DMARC Reports:
You can set up DMARC to send you a daily report from all participating email providers. The report shows:
* How often messages are authenticated
* How often invalid messages are seen
* DMARC policy actions that occur

Based on what you learn from the daily reports, you can refine your DMARC policy. For example, you can change your policy from **none** (monitor only) to **quarantine** to **reject** after you see that valid messages are being authenticated.

F-Secure's DMARC Record:
* `v=DMARC1;p=none;sp=none;rua=mailto:dmarc-rua@f-secure.com`

### What do the parameters mean?
* v - version
* p - action - If something doesn't pass DKIM or SPF, then do something (None, Quarantine, Reject - Use action “None” for feedback)
* rua - Aggregate feedback (Basic pass/fail data)
* ruf - Forensic Feedback (Specific headers of failed messages) - Optional field with the default value set to none
* pct - percentage (What percentage of failed messages do you want the rule to apply to) - Optional field with the default value set to 100%


The **none** definition essentially places DMARC into a test mode. ISP’s will check every message, but only send you reports instead of taking action on them. This allows you to start collecting details on your email sources before you do anything drastic. With SPF and DKIM, it is up to the ISP to decide what to do with the results. DMARC takes it a step further and gives you full control to set a policy to reject or quarantine emails from sources you do not know or trust, all based on the results of DKIM and SPF.

### What are DMARC Reports? 

ISPs who support DMARC will also generate reports on sending activity for your domain. The reports are XML files that are emailed to the email address specified in your DMARC record. The reports contain the sending source (domain/IP) along with whether the message passed or failed SPF and DKIM. This is one of the best aspects of DMARC. Not only does it allow you to control email security for your domain, it also gives you deep visibility into who is sending on your behalf AND if they are signing with DKIM or passing SPF.

## Useful Resources:
* [ZeroSec Blog](https://blog.zsec.uk/mail-tech-spf-pt1/)
* [SPF Checker](https://stopemailfraud.proofpoint.com/spf/)
* [OpenSPF](http://web.archive.org/web/20190224184030/http://www.openspf.org/SPF_Record_Syntax)
* [SMTP Security in a Changing World](https://www.youtube.com/watch?v=PHtukqtSdQc)
* [Postmark App](https://postmarkapp.com/guides)