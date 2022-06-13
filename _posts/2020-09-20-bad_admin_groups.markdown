---
layout: post
title: Bad Admin Groups
date: 2020-09-20 12:00:00 +0200
img: tmux.png # Add image post (optional)
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
tags: [InfoSec, Technical]
---

https://activedirectorypro.com/remove-local-admin-rights-using-group-policy/

# Can use Group Membership scanner and correlate member name with group name. (Kibana scanner)
# AD Groups that will form part of this:

#     Everyone
#     Authenticated Users
#     Domain Users
#     Domain Computers
#     Interactive
#     All Application Packages? - Need to research this group
#     Users
#     Guest
#     Anonymous
#     IUSR? - Need to research this group


# Effectively these groups don't require permissions so they should never form part of local admins.


# local_admin = S-1-5-32-544
# local_groups = S-1-1-0:Everyone
# S-1-5-4:Interactive
# S-1-5-7:Anonymous
# S-1-5-11:Authenticated Users
# S-1-5-13:Terminal Server Users
# S-1-5-14:Remote Interactive Logon
# S-1-5-17:This Organization
# S-1-5-32-545:Users
# S-1-5-32-546:Guests
# domain_prefix = S-1-5-21
# domain_suffixes = 501:Domain Guest
# 513:Domain Users
# 514:Domain Guests


AD groups which can be overlooked: (JIRA Bad administrator groups)
Everyone 
Authenticated Users 
Interactive - includes unprivileged users and over rdp with sspcred?
Domain Users
All application packages?
users
guest
anonymous
IUSR?


### Ensure that they are not in Local Admins

No need for them to be in local admins, if they are then bad job

### Look into ACLs - clear unnecessary privileges

# Within the <domain> domain, the DOMAIN USERS , DOMAIN COMPUTERS , EVERYONE and AUTHENTICATED USERS user groups were investigated to determine the rights associated with low privilege user account within the <domain>. It was found that the EVERYONE group provided all of the members within the group the GenericAll permission over 5 user accounts within the environment. This implies that all users within the EVERYONE group had the ability to compromise any of the following accounts:
# • SQLPRCLS
# • EMAILFAX
# • BVADMIN
# • PARTOLBMC
# • FAXEMAIL
# In addition, the EVERYONE group also had the Owns property over the PARTOLBMC AD object. In effect, any user within the <domain> domain would be able to take full control of the aforementioned accounts, and any permissions associated with those accounts.
