---
layout: post
read_time: true
show_date: true
title:  Essential security controls - Intro
date:   2021-10-30 10:00:00 +1000
description: A introduction article to a series on the essential security controls every organisation should implement.
img: posts/2021-10-30-essential-security-intro/hero.jpg
tags: [security,non-technical]
author: Peter Dodemont
---
This article serves as an introduction to a series I am writing around what I believe are the essential security controls that a business of any size should implement. This will by and large cover most of what standard's organisations and governments recommend around the world.
This article will list (and eventually link to) each of the controls I am writing an article about. It will also include a brief outline of what you can expect to read about in each article.
(My aim is to release a new article for this series every 2 weeks, so if you are interested in any of the below check back once every 2 weeks to see if there have been any new articles :) the controls are also not listed in any particular order, just in the order I thought of them)

## [Operating System Patching](\OS-Patching.html)
This should be pretty obvious to everyone, but there are still numerous reports coming out on compromises that happen because operating systems have not been patched. Having implemented this a several organisations in the last number of years. I will provide what I learned over the years and what has worked to not only patch in a timely manner but also in a way that minimises risk of widespread issues.

## [Least Privilege and Just-In-Time Access](\least-privilege-just-in-time.html)
Restricting privileges is another obvious one, mostly this is applied to not providing administrator access to users, but it actually reaches beyond this. This should also extend to other permissions within the business.

## ["Zero Trust"](\zero-trust-model.html)
Exploring the Zero Trust model and how you can get started with it.

## [Multi-factor Authentication](\mfa.html)
MFA has been on top of the list of items to implement for a number of years. In the past MFA was very intrusive, but these days there are very elegant ways to implement MFA. These not only provide additional security, they can also be implemented in a way that the user will very rarely even know MFA is being employed.

## [Email Filtering](\email-filtering.html)
Email filtering forms part of the first line of defence against a lot of threats. Good email filtering not only provides additional security, but it also increases productivity as users will no longer lose time by reading SPAM email. A single email might not take 1 person long to action, but multiple emails a day over an entire organisation costs a lot of productivity.

## [Endpoint Protection](\endpoint-protection.html)
With treat actors constantly evolving their tactics, good endpoint protection is essential. This not only covers antivirus software, but also anti malware, user behaviour and other items. I'll explore what I have done in the past and what I think is essential to do in the current day and age.

## Automation
This actually goes hand in hand with the Zero Trust model. Because by building automation you can remove the need for user accounts to have access to a lot of things. And security is only really a secondary benefit, the biggest benefit is increased productivity by not having to do boring and repetitive tasks. I will cover off how I decide when to automate and when to stop automating.

## Backups
Even if you implement all of the controls here, there is always a risk things can go wrong. A good backup solution will be essential in that situation so that you can recover quickly and effectively. With backup solution I am not really talking about any specific software, but rather about everything that encompasses backups be that retention polices, storage policies, testing procedure ... .

## Application Allow Listing
Application allow listing will stop the majority of malicious actors out there in their tracks. Sure there is groups out there who are able to leverage legitimate and built-in tools, but these are highly advanced actors. I will go through how I believe you can implement allow listing in a way that I believe balances usability and manageability with security.

## Application Patching
Like operating systems, application also need to be patched on a regular basis. This is a lot harder to do than operating systems, but there are some things we can do to minimise the risks and keep our applications as up to date as possible.

## Workstation and Server Hardening
With workstation and server hardening you are looking at ways to minimise the number of vectors any treat actor could exploit to get access. There are numerous hardening benchmarks out there to choose from, but blindly implementing them will probably lead to much frustration and ultimately to abandoning it completely. What we can do instead is target some safe areas to start with and then move through to harder items as we go.

## Credential and Certificate Management
When I say credential management most will probably think about password managers. And while they are very useful, I actually prefer to go down the less traditional road of SSO for credential management. Having it so a user only needs to remember 1 set of credentials will by way more likely to succeed than having to use another application.
Certificate management is something that is always a struggle. It is not only about making sure you know where your certificates are used and when they expire, it also covers who has access to them and who has accessed them. I will also cover my thoughts on what types of certificates you should be using.

## User Education
Users are your very last line of defence in most cases. And while we do everything to avoid the user ever seeing anything suspicious, they will more than likely see something at some point. I will cover what I believe is the most effective way of not only getting users educated but keeping them that way.

## Risk Management
While not something you can implement on a device, making sure you do proper risk management is crucial. Not only will it allow you to know where to focus on, it will also assist in providing better justifications for implementing the other controls on this page. Having information about what risk get most attempts of exploitation will allow you to put some hard numbers against the risks and compare the cost of doing nothing with the cost of implementing the appropriate control.

As always, feel free to reach out with any questions or comments.