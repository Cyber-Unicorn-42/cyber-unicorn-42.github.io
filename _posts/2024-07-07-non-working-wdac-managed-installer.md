---
layout: post
read_time: true
show_date: true
title:  The mystery of the non working managed installer WDAC policy
date:   2024-07-06 10:00:00 +1000
description: How I solved the mystery of the WDAC managed installer policy not working on some devices
img: posts/2024-07-07-non-working-wdac-managed-installer/hero.png
tags: [security]
author: Peter Dodemont
---
Some time ago I ran into a very strange problem and confounding problem: Windows Defender Application Control (WDAC) had been deployed and working for quite some time, including a custom-managed installer policy. Then during some application patching testing, I had a device where the policy didn't seem to be working.

## The Discovery

It all started with your average application patching process once you have deployed application control: Package app, test deployment in audit mode, test deployment in enforced mode, look at the logs, fix application control policies, test in enforce mode again and finally repeat the last two steps until it's working.

The first 3 steps happened the same as usual, but in step 3 I started noticing something odd. The application patch being installed was being blocked and the initiating process was listed as "microsoft.management.services.intunewindowsagent.exe" (this is the process used by the Intune Company Portal application to do software installs). I thought this should be impossible as we have a managed installer policy configured that marked this binary as a managed installer, so whatever file is executed by it should be allowed.

## Down The Rabbit Hole

Like any good start to troubleshooting, I asked someone else to also try installing this application through the Intune Company Portal as well, and lo and behold they were experiencing the same issue. There was an added wrinkle though, this 2nd person was able to install several other applications through the Company Portal without any issues.
This pointed to the issue being specifically related to this application and it somehow not meeting some criteria causing it to not be processed by the managed installer policy.

I validated that the managed installer policy was running, not only by checking the remediation deploying it through Intune, but also validating it was the effective policy on the device, all looked good. (I will be publishing an article here on how I'm doing the remediation, which I'll link here). I looked up to see if the file hashes were being specifically blocked, but nope. How about maybe the certificate the file was signed with, nope the file was not signed at all. Then I looked up the file new to see if that was anywhere in the policy, nope the filename was also nowhere to be found.

When I eventually ran out of things I could think of to check, I advised to reset this device as it must be having some corruption of some system files. When the reset then failed to work, this seemed to confirm my theory and I stopped looking further into it.

## The Solution

Eventually, the person came back to me saying that the issue was still occurring even after they rebuilt the device using the install media. Now I was really perplexed, as it seemed extremely unlikely that there would be corruption after a clean rebuild.

Fortunately, I had another more pressing WDAC issue I needed to work on. I jumped on a call with some colleagues and started looking at the issue they were having. Lo and behold as soon as I looked at the log it was exactly the same issue.
These colleagues weren't as familiar with WDAC, so I started to explain how it all fits together. That is when the magic happened. As I was explaining the managed installer, I remembered that the managed installer policy is not really a WDAC policy, it is using AppLocker functionality. I started going down the path of seeing what could be causing AppLocker to not be functioning correctly. Came across some info that some editions of Windows didn't support AppLocker if you were running versions earlier than 2004. I was pretty sure we were on the most recent version, but since I was still on the call I asked the user to double-check. They confirmed we were on the most recent version.

After a bit more digging around the deployment manual for AppLocker, I came across a section that talked about a specific service that has to be running for AppLocker to function properly, this service was the "Application Identity" service (AppIDSvc).
The call had finished by that point so I asked my colleague over chat to see if they could check if that service was running, it wasn't running. I told them to start the service and then try the install again. And as if by magic, the application installation was now working.

## Fixing The Problem

Now that we knew starting the service solved the issue, I started the process of trying to understand why the service wasn't running. The page where I found out about the service mentioned the service was set to manual by default and it was recommended to set it to automatic. I decided to check the service on 2 devices I had access to. On both devices, the service was set the manual (trigger start). Now why I never had any issues with software installs is a little confounding, the only explanation I can think of is that whatever trigger causes the service to start is happening often enough the service was running when I have done installs.

Finding the service set to manual on my device I went looking for some config related to this service in our deployment setting for WDAC. I couldn't find anything at all referencing this service.
I knew I had to put in a fix that would work across the board, but the instructions page talked about using group policy for setting the service. This was not possible for me, as all the devices are cloud-only. I decided to fix the issue by putting in a new remediation via Intune that would set the service to automatic (I plan to create a post of this as well, which I'll link here).


While this looks like a fairly quick and straightforward process all written out here, it took me several weeks to figure this out. Not just because there doesn't appear to be any information on this I could find, but also because I had a number of other things to work on during that time as well.
It was great troubleshooting, which I haven't had to do in a while. It reminded me of this line from the Stargate SG1 episode [Learning Curve](https://www.gateworld.net/sg1/s3/learning-curve/): "Figuring something out on my own, especially if it's been eluding me a while. It's far more satisfying than having the answer given to me."

That being said, if you do have questions or comments, feel free to reach out to me.