---
layout: post
read_time: true
show_date: true
title:  Endpoint Protection for the current day and age
date:   2022-03-19 10:00:00 +1000
description: What I think endpoint protection in the current day and age should like.
img: posts/2022-03-19-endpoint-protection/hero.jpg
tags: [security,non-technical]
author: Peter Dodemont
---
Endpoint protection used to be as simple as putting antivirus software on a device and making sure it was up to date. But these days malicious actors are much more adept at evading antivirus software. Not only do they add evasion techniques into their applications, they also use legitimate application that are already present on the system like for example PowerShell. So just having antivirus software running on the device and scanning once every day or week is no longer enough.

## Defining an Endpoint
I'll start with what I define as an endpoint: an endpoint is any device (physical or virtual) that can be accessed by a single or multiple users. This means that the PC, laptop, or tablet a user uses is an endpoint, but so is a server or the mobile device a user might be using. While I define all of these as endpoints, that doesn't mean that I implement the same solution for any of the items listed below. There are solutions that can work across multiple endpoint categories but generally they won't cover all of them. And there are certain scenarios where implementing the recommendations below might be more difficult (e.g. BYOD), but there will usually be a way to at least partially implement them.

## Endpoint Protection Software
It used to be called antivirus software, but it has over the years evolved to do so much more than just scan for viruses on your device. This is why we call it endpoint protection software these days. I'll go through what I believe are the most important features one by one and tell you why I think they are important and why you should consider them for any new Endpoint Protection software you want to implement.

### Anti Malware and Antivirus
You might wonder what the difference is between both of these as they are used interchangeably quite a bit. Traditionally viruses are application that have a destructive nature (e.g. ransomware), malware is a global term for any application with a malicious intent. Malware includes viruses, but it also includes applications that steal your credentials or log you screen and keystrokes, as well as Office documents with malicious macros.

I think it goes without saying that you are going to want to have an endpoint protection solution that is able to detect and stop any known malware applications. And while there is plenty of malware that can evade some of the endpoint protection solutions, there is much more that can't. This mostly comes down to whether the tools is available as a commodity for anyone to buy and use, or if the tools is custom developed by the malicious actor and kept for their own use.

### Ransomware Prevention and Detection
With ransomware being the most prevalent threat currently and there not being any signs of it slowing down, I feel this category of malware deserves a special mention of its own.

When I talk about ransomware prevention, I'm mostly talking about protections to limit the amount of files that can be changed in a given period or a given folder or stopping known ransomware from running. An example of the protections is Windows Controlled folder access, it allows you to protect specified files and folders from being changed by any app that is not listed as trusted.  
When talking ransomware detection, I'm mostly talking about the ability of the software to detect changes to the system that indicate ransomware is running. This usually involves detecting large amounts of files being changed by a single application.

You might think that if you have the prevention in place, then you won't need the detection. But there is no guarantee that the prevention won't be circumvented. Someone could find a way to exploit a trusted app or they might find a way to disable the software providing the protections or ... . having both means that you are provided greater coverage for more scenarios.

### User Behaviour Analytics
As mentioned earlier there is more and more malware out there that is able to circumvent traditional detection either by the way the application is written or by using legitimate tools already present on the device. User behaviour analytics (UBA) is a way to try and catch these types of applications. In general terms the software will collect large amounts of telemetry from the device about how the user normally used their device. E.g. what time of the day do they normally work, what applications do they use, and when. The amount and specific information collected will differ between tools, but they all have the goal of building a profile of the usage of the device by that specific user. Once they have that profile, they can compare future action against the profile and see if is expected or not. Let me give a simple example, PowerShell might run on a user's device with some regularity as it is commonly used for automated tasks by device administrators, but if all of a sudden there is a PowerShell command that is running an obfuscated command, then UBA can pick this up. Another example could be, that normally the user works 9am to 5pm, then one day some activity is happening on the device at 2am. Again, this would be picked up by UBA. In reality the process is more complicated as there could be legitimate reasons for the user working at 2am (e.g. they travelled to a different time zone), but the concept is the same.

UBA are a great way to detect applications or application executions that would normally go unnoticed. And the more detailed the information is the better profile you can build. But as with all of the other recommendations, there are also ways malicious actors could get around this, if they have access to the device, they can build their own profile and blend in their actions, or they could try and influence the profile. That being said, those techniques are most likely to be used by highly skilled malicious actors. There are far more malicious actors out there that are more opportunist, and attacks by those can be thwarted much more easily if you have UBA at your disposal.

## Device Compliance
Device compliance is an important part of endpoint protection. Its great having all these tools implemented, but if they are not running, then they serve no purpose. This is where device compliance comes into play, it will evaluate a device to confirm that the tools are running and are configured the way you want them to. Device compliance can be part of the endpoint protection software, or it can be evaluated outside of it. Microsoft Endpoint Manager (MEM) for example as a section for compliance policies that will evaluate a number of items (e.g. does the set password meet complexity rules, is the firewall on ...) and if any of the checks fail the device will be marked as not compliant. You can then using other parts of MEM and Azure to take action of that status (e.g. prompt for MFA or block access).
Generally, each piece of software will have its own way to monitor the compliance of all the devices under it's management, and it is important to keep an eye on that compliance and action any non-compliance accordingly.

By using device compliance, you can prevent malicious actors from being able to access your environment from one of their own devices. This means that they will need to transfer any tools they want to use to one of your devices, which increases their risk of being detected. And the having a greater chance of detecting malicious actors is always a good thing.

## Endpoint Detection and Response
The speed at which you are able to react to any of the signals picked up is crucial, the faster you can respond to detections the less likely there is to be major outages. Which means that unless you work for a large organisation or a security vendor, it's unlikely you have a dedicated security team that is available 24/7/365 (and they need to be 24/7/365 as malicious actors prefer to launch their actions in or very close to the off-hours of the business). Endpoint Detection and Response (EDR) can solve this for you, by either providing automated response to any detection or by having a team of people available to action the detections for you.

While automated response sounds like it would be ideal, I prefer to have a team of people available (and if you have a team, they will usually have automated tools are their disposal anyway). The reason for preferring a team, is that the right team is much better at determining if unusual behaviour is actually malicious or not. They are able to make decisions on patterns that have never been seen before. Automated responses on the other hand will rely on known patterns with known solutions. If the solution fails, there isn't anything the automated solution can do. Don't get me wrong automation is great for getting mundane tasks out of the way, but if there is something new that has not been seen before automation is of no use.

These are my top recommendations for when you are looking into endpoint protection software. There are other aspects to endpoint protection that I have not covered here (e.g. application allow listing), but I feel they deserve a separate article as there is quite a bit to talk about on those topics.

As always, if you have any questions or comments, please reach out.