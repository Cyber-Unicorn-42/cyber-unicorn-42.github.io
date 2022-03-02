---
layout: post
read_time: true
show_date: true
title:  Troubleshooting Intune/Endpoint Manager on Windows Devices
date:   2022-02-22 10:00:00 +1000
description: How can you troubleshoot all the different items you can apply or install on Windows devices through Intune/endpoint manager on the endpoint.
img: posts/2021-10-02-troubleshooting-intune/hero.png
tags: [intune/endpoint-manager]
author: Peter Dodemont
---
After many years of working with SCCM I have become very comfortable with troubleshooting most issues that arise from anything deployed through SCCM. These days most of the items I deploy are through Intune/MEM. This means I have had to start learning how and where I can do troubleshooting for items deployed through Intune/MEM. I will use this post to catalog where I do all my troubleshooting. I will update this post as I learn more about how and where to troubleshoot deployments through Intune/MEM.

* [Configuration Polices](#config-policies)
    * [Error 404](#conf-pol-404)
    * [Error 864](#conf-pol-864)
* [Win32 App Deployments](#app-deployments)
    * [PowerShell Note/File Not Found](#powershell-note)
* [Proactive Remediations](#proactive-rem)
* [Scripting](#scripting)
* [Company Portal](#company-portal)
* [Windows Information Protection](#wip)
* [Tools](#tools)
    * [CMTrace](#tools-cmtrace)
    * [Windows Calculator](#tools-calc)

## <a name="config-policies"></a>Configuration Policies
Troubleshooting configuration policies is done through the event viewer. The main log to look at, is the "Admin" log under "Applications and Services Logs -> Microsoft -> Windows -> Microsoft-Windows-DeviceManagement-Enterprise-Diagnostics-Provider".
This log contains errors from any configuration policies that you have applied to the device or user. There are a variety of errors, below are the ones I have had to deal with so far.
* <a name="conf-pol-404"></a>Error 404
![Config Policy Event Viewer 404 Error](/assets/img/posts/2021-10-02-troubleshooting-intune/config-policies-eventvwr-404.png "Config Policy Event Viewer 404 Error")
This particular error I just ignore. The error seems to point to the loading of an ADMX file, but the "ADMX file" cannot be found. Every time I have checked, the particular ADMX referenced has been loaded correctly and I have no issues applying settings from that ADMX.
* <a name="conf-pol-864"></a>Error 864
![Config Policy Event Viewer 864 Error](/assets/img/posts/2021-10-02-troubleshooting-intune/config-policies-eventvwr-864.png "Config Policy Event Viewer 864 Error")
This error indicates that a particular setting cannot be found in a custom config policy (this is where you have first loaded a ADMX file and then applied setting from that ADMX through OMA-URI's). In this particular example the "CDPF_DisableConnectedPDT" setting cannot be found in the Foxit Phantom PDF ADMX I loaded.

Changes to config policies will trigger Intune/MEM to push an update to the device, but if that fails for some reason you can manually force an update by running a sync from the Company Portal app.

## <a name="app-deployments"></a>Win32 App Deployments
The "IntuneManagementExtension.log" file is where you can do the troubleshooting for app installations. You can view this log with any log viewer or text editor, personally I use [CMTrace](#tools-cmtrace). The file is located at "C:\ProgramData\Microsoft\IntuneManagementExtension\Logs" (unless you changed the location of ProgramData). When first opening the log, it can look daunting as there is a lot of information in it. Once you understand it's make-up though, it becomes a lot less intimidating.
I do most of the heavy lifting for app deployment troubleshooting in my script, I cover that off in more detail in the [scripting](#scripting) section, but even without it you can still get useful information from the log.
Mainly what you are looking for is entries prefixed with [Win32App]. This indicates that any log sections following it are related to Win32 App installs. Until you reach a new line that has a prefix (this could be the [Win32App] prefix again or something else).
The first step you can take is to do a quick search for either the GUID or a distinct section in the name of the app you are installing. Each app install section will begin with a line similar to the below which contains both the name and the GUID of the app.
```
[Win32App] ExecManager: processing targeted app (name="'Microsoft Azure Information Protection 2.5.33.0', id='c7f686fd-a49a-497b-aed0-4146ea1b2abe') with intent=3, appApplicabilityStateDueToAssginmentFilters= for user session 1
```
If you don't have the GUID already, it would be good to take note of it now, as most parts of the log refer the GUID and not the name.
First a check is performed to see if there are any dependencies. If there are dependencies, a order of operations is created and the dependencies are added to the list of apps that need to be evaluated.
Then, a number of steps are performed for each app, namely: detection, applicability, extended requirements, download, execution and detection again. Each step starts with
```
[Win32App] ===Step===
```
Detection is always performed as the app might have been installed or uninstalled manually prior. Each detection rule defined for the app will be processed in order and if one of them fails the application detection will be marked as failed. Below is an example of a failed detection from an MSI product code.
![Failed MSI Detection](/assets/img/posts/2021-10-02-troubleshooting-intune/failed-msi-detection.png "Failed MSI Detection")

On the first line in the screenshot you can see "SideCarProductCodeDetectionManager", this will change depending on what type of detection you have configured.
After detection, applicability is checked. Applicability refers to the requirements that are available in the interface as per the below screenshot.
![Applicability Rules](/assets/img/posts/2021-10-02-troubleshooting-intune/applicability-rules.png "Applicability Rules")

Following applicability, extended requirements are checked. This refers to any additional requirements you configured on the app. See the section highlighted in the screenshot below.
![Extended Requirements Rules](/assets/img/posts/2021-10-02-troubleshooting-intune/extended-reqs-rules.png "Extended Requirements Rules")

The next step is to check if the application has already been downloaded. If it hasn't or there is a new version available the download will be started and stored in "C:\Program Files (x86)\Microsoft Intune Management Extension\Content\Incoming" (unless you changed the location of the Program Files (x86)) and the file will be named with the GUID of the app followed by "_x.bin", where "X" is equivalent to the version of the intunewin file of the app.
Then we have the execution step. Like with the download there is check. In this case the check is to see if installation of that version of the app has been attempted in the last 24 hours already, if it has the app will not attempt another install.
This section is where most of the errors will occur. The log will show a line showing the install command
![Win32 Install Command](/assets/img/posts/2021-10-02-troubleshooting-intune/win32-execute.png "Win32 Install Command")

A few lines later there will be a line with an exit code. In this example the exit code is 1
![Win32 Exit Code](/assets/img/posts/2021-10-02-troubleshooting-intune/win32-exitcode.png "Win32 Exit Code")

I have not defined the exit code as failed in the application properties, but if I did it would come up as failed. And with some additional work upfront while writing install scripts you can precisely pinpoint where an error is occurring (see the [scripting](#scripting) section for more info).
Finally, another detection is performed to confirm that the application was installed correctly.

Available and required applications are updated on a schedule, but you can force an update by doing a sync in the Company Portal app.

* <a name="powershell-note"></a>PowerShell Note / File Not Found
A quick note about the PowerShell environment used by Win32 apps. The PowerShell environment runs as **32 BIT**. I found this out the long and hard way when trying to run an application that resides in System32 and getting a file not found error. It took me several days to finally figure it out. The error was 100% correct but I was unable to replicate it locally as interactively PowerShell runs in 64 bit.

## <a name="proactive-rem"></a>Proactive Remediations
Proactive remediations log their output to the "IntuneManagementExtension.log" file. You can view this log with any log viewer or text editor, personally I use [CMTrace](#tools-cmtrace). The file is located at "C:\ProgramData\Microsoft\IntuneManagementExtension\Logs" (unless you changed the location of ProgramData). Proactive Remediation log entries start with the prefix [HS].
The process for proactive remediations is really simple. First the detection script is run and then the remediation script. Both steps follow the exact same process: the relevant script is executed and the last line from the output is captured and displayed in the log.
Unlike with Win32 Apps, proactive remediations are only referenced by their GUID in the log. This means first we have to get the GUID.I use the following method for getting the GUID, but you can do whatever works for you. I right click the link of the proactive remediation in the Intune/MEM Portal and copy the URL
![Proactive Remediation URL Copy](/assets/img/posts/2021-10-02-troubleshooting-intune/proactive-rem-url-copy.png "Proactive Remediation URL Copy")

And from the URL I can get the GUID. It's located right after the "/ID/"
![Proactive Remediation URL GUID](/assets/img/posts/2021-10-02-troubleshooting-intune/proactive-rem-url-copy.png "Proactive Remediation URL GUID")

With the GUID I can now identify the section of the log that is relevant to that specific proactive remediation.
Proactive remediations only have 2 exit codes they accept 0 for success and 1 for failed. The below example looks for a specific print driver on the system, if the driver is found remediation needs to be run. The detection script exits with an exit code of 1 indicating the driver was found. This can also be seen in the last line outputted by the script as it is captured in the log.
![Proactive Remediation Detection Failed](/assets/img/posts/2021-10-02-troubleshooting-intune/proactive-rem-det-fail.png "Proactive Remediation Detection Failed")

The remediation script is run straight after and its log entries are identical. It also shows you the error code as well as the last line on the output before the script exited
![Proactive Remediation Remediation Failed](/assets/img/posts/2021-10-02-troubleshooting-intune/proactive-rem-rem-fail.png "Proactive Remediation Remediation Failed")

Showing the last line of the output gives us the ability to capture the exact error as long as we have proper error handling in the scripts. I cover this in the [scripting](#scripting) section.

To trigger a run of the proactive remediation what you need to do is restart the "Microsoft Intune Management Extension" service. Once the service has been restarted It will check if any new or updated remediations are available and if some are found they are scheduled to be executed 5 minutes after the service was restarted.
![Proactive Remediation Scheduler](/assets/img/posts/2021-10-02-troubleshooting-intune/proactive-rem-scheduler.png "Proactive Remediation Scheduler")

## <a name="scripting"></a>Scripting
I write all my scripts in PowerShell as it is very flexible and easy to use, so that is the only language I will cover here.
One of the most important things you can do to make troubleshooting easier is to write error handling into your scripts.
Error handling in PowerShell is incredibly easy, all you need to do is use a Try-Catch block and the rest is handled by PowerShell internally.
You put the commands you want to execute in the Try section and then you put the actions to take when an error happens in the Catch section. One important thing to note is that the error needs to cause the command to stop proceeding, you can force this on most cmdlets in PowerShell by using "-ErrorAction Stop".
In the catch block in the example below, you'll notice that I captured the error message in a variable so that I can provide a custom error message as well as include the actual error message returned.
```powershell
Try {Remove-Item -Path C:\Temp\test.txt -Force -ErrorAction Stop}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "File Removal Error: $ErrorMsg"
    Exit 1
}
```
When put in a script and run it gives the below output
![Scripting Error Text](/assets/img/posts/2021-10-02-troubleshooting-intune/scripting-error-text.png "Scripting Error Text")

And upon checking the last exit code, you will notice it is 1
![Scripting Error Code](/assets/img/posts/2021-10-02-troubleshooting-intune/scripting-error-code.png "Scripting Error Code")

For Win32 Apps you can specify whatever error number you want. Since the log does not contain any output other than the error number, I suggest you divide the script into sections and provide a unique error code for each section. This way you can easily identify where the error occurred. The error code will also be passed to Intune/MEM so you can retrieve it from the installation status. In Intune/MEM it will be in hex and prefixed with "0x8007".
For proactive remediations the error code must be 0 or 1, 0 indicates success and 1 indicates failure. But since the last line outputted to the console is shown you can provide a custom error message for each section to assist in pinpointing where the error occurred.

## <a name="company-portal"></a>Company Portal
I wrote a separate article on how to clear stuck items from the company portal which you can find [here](/clear-company-portal-status.html).
The main point from that article is that the company portal retrieves its information from a specific key in the registry. That key and its sub keys can be used to clear stuck items and find error messages and results. Below is the key in question
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\IntuneManagementExtension
```

## <a name="wip"></a>Windows Information Protection
If you are using windows information protection you will almost certainly come to a point where you will want to force and sync and confirm that the policy has applied. You can force the sync of the policies by opening the Windows Settings app and going to Accounts -> Access work or school -> Click on "Connected to *yourdomain*" -> Info -> Sync.
Once you have done that and it completes, you can go to "C:\Windows\System32\AppLocker\MDM\ _RandomNumber_\ _GUID_\AppLocker\EnterpriseDataProtection" (where _RandomNumber_ and _GUID_ are unique to your device and environment).
Depending on your configuration you will see up to 4 folders: 1 for allowed desktop apps, 1 for exempt desktop apps, 1 for allowed store apps and 1 for exempt store apps. Each will contain yet another folder, and once you open this folder you will see a file called "Policy" open this file with a text editor and it will contain the details of the configuration, matching what you entered in Intune.

## <a name="tools"></a>Tools
* <a name="tools-cmtrace"></a>CMTrace

CMTrace is the log viewer that is included with SCCM. It is a tool not just for looking at the SCCM logs, but also for any logs generated by Intune/MEM. You can find more information about CMTrace [here](https://docs.microsoft.com/en-us/mem/configmgr/core/support/cmtrace).
The tool is provided with SCCM installs or you can download and install the "System Center 2012 R2 Configuration Manager Toolkit" from [here](https://www.microsoft.com/en-au/download/details.aspx?id=50012).
Since CMTrace is a self-contained executable, I keep a copy of it in my tools folders for easy access in the future.

I hope you now feel more confident in troubleshooting any issues that arise when deploying items through Intune/MEM.
As always please feel free to reach out to me with any questions.

* <a name="tools-calc"></a>Windows Calculator

You are probably wondering why I would put the builtin Windows calculator as a tool. This is because you can use it to easily convert the status codes provided in the SCCM logs to the actual error messages. By status codes I'm talking about those that start with "0x8007". Most of the time they are just hex values that can be converted easily. But I have come across the odd occasion where this doesn't work very well or where I need to subtract the "8007" that gets automatically added to each status code. To be able to do the conversion all you need to do is switch the calculator to the "Programmer" mode from the menu.