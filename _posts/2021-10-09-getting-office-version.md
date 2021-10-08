---
layout: post
read_time: true
show_date: true
title:  Getting Office 365 Client version in Intune/MEM
date:   2021-10-09 10:00:00 +1000
description: How to can you get the Office 365 version for all your devices through Intune/MEM.
img: posts/2021-10-09-getting-office-version/hero.png
tags: [Intune/Endpoint-Manager,PowerShell]
author: Peter Dodemont
published: true
---
I recently had a need to get the Office 365 version from all of the devices in my org. The reason was that we previously had to stay at a specific Office 365 version as it caused issues with an in-house developed app (this was done using the Intune/MEM target version policy). Improvements have been made to the app that meant we could now start upgrading Office 365 to the most recent version and keep it automatically updating. You would think that this is something that can be easily achieved through built functions in Intune/MEM, but there is no built-in report (that I could find) that provides this info. Discovered apps only lists some Click-to-Run components (which are different to the actual Office 365 app versions when you have a target version specified) and the device install status of the deployed app also doesn't list the actual version installed. I also looked at the available analytics reports but again no luck there. This meant I would have to write a script that will collect the version and then provide it to me somewhere. Since I actually wanted to start using that as an ongoing metric to keep track of compliance within the business, I opted to use my current favorite feature: Proactive remediations.

## Why Proactive Remediations?
You might think using proactive remediations is odd as there is nothing to remediate, but clearly this use case is something Microsoft had in mind when they developed this feature. I say this, because there is no requirement to actually provide a remediation script, the only script required is the detection script.
Another benefit of using proactive remediations is that the output is available directly searchable in the Intune/MEM and it is also exportable. This makes it easy to find a specific device right there and then, but also gives the ability to perform more advanced checking by exporting the results if needed.

## Reporting The Office 365 Version
There are several ways you can go about getting the version of Office 365 installed. The easiest are to look at the version of one of the executables or to look at a specific registry entry. I opted for the registry entry as that seemed just as reliable as getting the version directly from the executable and I already had a proactive remediation script that looked at registry entries that I could tweak.

## The Script
The script is pretty straightforward, you can find it [here](https://github.com/PeterDodemont/Scripts/blob/main/Intune/ProactiveRem-RegValue-Reporting.ps1).
The script contains a section that allows for checking values in the current user hive of the currently logged in user (even when running under system), but that is not relevant here so I'm going to skip over it.
First, I set a variable with the full path to the registry value I want to check.
```powershell
$RegKeyFullPath = "HKLM:\SOFTWARE\Microsoft\Office\ClickToRun\Configuration\VersionToReport"
```
This is followed by splitting that path out into the registry key and the registry value.
```powershell
# Get the parent and the leaf from each path
$RegKeyPath = Split-Path $RegKeyFullPath -Parent
$RegKey = Split-Path $RegKeyFullPath -Leaf
```
Next, I get the actual data of the registry value.
```powershell
# Get registry key value
$RegKeyCurrentValue = Get-ItemPropertyValue -Path $RegKeyPath -Name $RegKey -ErrorAction Stop
```
Then I write out the value and exit the script with the success exit code.
```powershell
# Report registry key value.
Write-host "$RegKeyCurrentValue"
Exit 0
```
You never know what oddity might be out there, so the whole thing is wrapped a Try Catch block for error handling just in case.

This is all there is to it. As always if you have any questions or comments feel free to reach out.