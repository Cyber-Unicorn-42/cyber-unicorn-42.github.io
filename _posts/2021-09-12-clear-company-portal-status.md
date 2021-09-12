---
layout: post
read_time: true
show_date: true
title:  Clearing the status of an Endpoint Manager deployed app
date:   2021-09-12 10:00:00 +1000
description: How do you clear a downloading or installing status in the company portal.
img: posts/2021-09-12-clear-company-portal-status/hero.png
tags: [Intune,CompanyPortal,EndpointManager]
author: Peter Dodemont
---
If you are reading this you have probably come across an issue where an Microsoft Endpoint Manager (aka Intune and referred to from now on as MEM) deployed app has gotten stuck in a particular state most commonly downloading or installing. And while there is no why to clear this from the GUI there is a way to clear the status.
First thing you are going to need is the GUID of the application that is stuck. I have found the easiest way to get the GUID is by going to the MEM portal and into the application details. The final part of the URL will then be the GUID of the application.
![MEM/Intune app GUID](posts/2021-09-12-clear-company-portal-status/intune-app-guid.png "MEM/Intune app GUID")
Once you have the GUID you need to open the registry editor and go to
'''
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\IntuneManagementExtension\SideCarPolicies\StatusServiceReports\UserOrDeviceGUID
(The "UserOrDeviceGUID" is the Azure GUID of the user or device that the app has been deployed to)
'''
Here delete the key that matches the GUID of the application that is stuck. You can look the UserOrDeviceGUID if you want, but in most cases there is only a limited number of GUID there and it is more efficient to just look through them quickly.
![Regedit Status Clearing](posts/2021-09-12-clear-company-portal-status/regedit-sidecarpolicies.png "Regedit Status Clearing")
You will notice that one of the values underneath the application GUID key is named "Status" and that this value will match the status in the company portal.
Once you delete the entire key, just restart the company portal an the app will not present as failed and you can re-launch the install.

For good measure you can also go to
'''
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\IntuneManagementExtension\Win32Apps\UserOrDeviceGUID
'''
and delete key with the application GUID there as well. This clears the compliance status of the app, but that hasn't been something I have found to be necessary.
![Regedit Compliance Clearing](posts/2021-09-12-clear-company-portal-status/regedit-win32apps.png "Regedit Compliance Clearing")

If you also want to force a re-download of the application files you will need to go to
'''
C:\Program Files (x86)\Microsoft Intune Management Extension\Content\Incoming
'''
In that folder delete the file where the name starts with the GUID of the app in question.
The _x is represents the version of the file uploaded into MEM. It increments by 1 every time you upload a new file for a specific app.

That is all there is too it, as always if you have any questions please feel free to reach out.