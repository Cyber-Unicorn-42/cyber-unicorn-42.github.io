---
layout: post
read_time: true
show_date: true
title:  Updating Group Membership Without Restart or Logoff
date:   2022-03-25 10:00:00 +1100
description: How can you update someone's group membership without restarting or logging off.
img: posts/2022-03-05-update-groups-no-restart/hero.png
tags: [intune/endpoint-manager]
author: Peter Dodemont
---
We a lot of people no longer working in the office, getting them to do a restart to update group membership is no longer a feasible option. As they are not on the company LAN, they won't get the updated group memberships during a restart. But there are still ways of doing this for both users and devices.

## The Problem
When a user's or device's group membership is changed on the on-prem Active Directory (aka AD), these changes don't flow through automatically to the device. Even if the device is connected to the corporate network either directly from an office or through a VPN. When the user or device is on the corporate LAN, updating the membership is as easy as restarting the device or logging off and on. Unfortunately, this doesn't work for people working remotely as the VPN might not have established yet at the logon of the user, thus the device will have no way of communicating with the domain controller to get the updated group membership details. Fortunately, there is a way to refresh the group memberships without a restart or a log off, by clearing the Kerberos ticket and re-acquiring a new one.

## The Solution
While previously we could just follow the same process for both users and devices (i.e. restarting the device). Clearing the Kerberos ticket without a restart requires different steps to complete for each. Let's start with the process for the user as that is probably the most commonly used.
* The User Process

The process is really straight forward and only has 2 steps.
First, you need to open a command prompt as the user and run the below command.
```Batchfile
klist purge
```
This command deletes your Kerberos ticket.
To get a new ticket all you need to do is connect to a network resource, this can be a network drive or you can browse to the root of the domain or anything else that will initiate a connection with the domain.
That is all that is required.

If you want to confirm that this worked, you can use the below command before and after and check the value of "StartTime" has changed.
```BatchFile
klist tgt
```
![Kerberos Ticket Starttime](/assets/img/posts/2022-03-05-update-groups-no-restart/kerb-ticket.png "Kerberos Ticket Starttime")

* The Computer Process

The process for doing it for a computer is also only 2 steps.
First, run the below command from an elevated command prompt.
```batchfile
klist -li 0x3e7 purge
```
Similar to the command for the user this clears the Kerberos ticket, but this time for the device. The local device will always have 0x3e7 as the LogonId (the LogonId for the user changes, but since the purge command runs under the current user context by default, we don't need to specify one with the user command).
Once you have cleared the ticket all you need to do is run a gpupdate by running.
```Batchfile
gpupdate /force
```

Just with the user you can also confirm this worked by comparing the start time of the Kerberos ticket by running the below command before and after.
```Batchfile
klist -li 0x3e7 tgt
```
![Device Kerberos Ticket Starttime](/assets/img/posts/2022-03-05-update-groups-no-restart/kerb-ticket-device.png "Device Kerberos Ticket Starttime")

And that is all there is to it. As always if you have any comments or questions, please reach out.