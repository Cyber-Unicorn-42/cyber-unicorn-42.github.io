---
layout: post
read_time: true
show_date: true
title:  Setting file and folder permissions through PowerShell
date:   2035-01-01 10:00:00 +1000
description: How to set file and folder permissions using powershell including inheritance and propagation.
img: posts/2021-12-04-powershell-permissions/hero.png
tags: [powershell]
author: Peter Dodemont
---
There is plenty of information out there on how to set permissions via PowerShell, but most of it does not seem to include much information on how to set propagation and inheritance. I will focus on those two items in this article. I will cover off how to set the permissions as well, so you can just have a single source to refer to should you need it.

## Propagation and Inheritance
Before getting into how to set the permissions I want to give a quick refresher on what these 2 properties are, what they do and how they are linked.
Inheritance indicates if that particular access rule is allowed to be passed on the any child items, and which item types it can be passed onto to. Access rules are just the individual entries that make up the access control list (or ACL). They are also referred to as access control entries (or ACE). PowerShell refers to them as access rules so that is what I'll be using. Inheritance is only available on folders (since files do not have any child items). There are two options for inheritance:
* Subfolders: Indicates that the access rule can be set on any subfolder
* Files: Indicates that the access rule can be set on any files
Propagation on the other hand indicates whether or not the access rule applies to the folder it is set on.
The inheritance and propagation details of the access rule can be found on the advanced properties windows of the item in the GUI.
![Inheritance in the GUI](/assets/img/posts/2021-12-04-powershell-permissions/inheritance-propagation-gui.png "Inheritance in the GUI")
By combining the option from 2 properties you can get control over where the access rule will be applied. The most common is "This folder, subfolders and files" as that allows it to be set on the folder and any child item. But it is not uncommon to use just "This folder" if you want a particular permission on a single folder. "Files" is also used frequently if you want to prevent users from changing the folder structure but allowing them to add, remove or modify files.
This means that getting these right when setting permissions via PowerShell is just as important as setting the actual permissions. Even more so because the defaults in PowerShell are different from the defaults the GUI uses. The GUI defaults to full inheritance and enables propagation (i.e., this folder, subfolders and files). PowerShell disables inheritance but does have propagation enabled.

## Setting The ACL
Setting the ACL is not overly complicated. It is only a handful of lines.
First, I grab the existing ACL from the item.
```powershell
$ACL = Get-Acl -Path C:\temp
```
Next, I create a new access rule to be added to the ACL. This is where the propagation and inheritance come in. As you can see from the line below there are a number of properties that you need to define when setting the access rule. The properties need to be in a particular order, and some are optional. In order they are:
* The name or SID of the user or group the access rule will apply to.
* The particular permission the access rule will apply. See [here](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesystemrights?view=windowsdesktop-5.0) for a list of possible values.
* The inheritance of the access rule. This is an optional property.
* The propagation of the access rule. This is also an optional property.
* The type of access rule. This is either Allow or Deny.

You'll notice that the propagation and inheritance properties are numeric values. You can find information on which values to use [here for propagation](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.propagationflags?view=windowsdesktop-5.0) and [here for inheritance](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.inheritanceflags?view=windowsdesktop-5.0). The values you will find on those pages need to be added together if you want to provide the settings. E.g. if you want to set inheritance to be just for subfolders you set the inheritance property to 1. If you want have inheritance for subfolders and files you set the property to 3 (as I have in my example below).
Note that if you do not supply a value for both propagation and inheritance, they will use 0. This means there is no propagation or inheritance for that access rule.
```powershell
$AccessRule = New-Object System.Security.AccessControl.FileSystemAccessRule("Users","Modify","3","0","Allow")
```
Then, you add the access rule to the set of access rules already in the ACL
```powershell
$ACL.SetAccessRule($AccessRule)
```
And finally, you write the ACL back onto the file or folder.
```powershell
$ACL | Set-Acl -Path c:\Temp
```
All together this will look like the below. As with all my other scripts it's wrapped in a Try-Catch block for error handling. I also generally use variables or parameters so I can easily re-use the script in the future.
```powershell
$ACLPath = "C:\Temp"
$Identity = "Users"
$FileSystemRight = "Modify" 
$Propagation = "0"
$inheritance = "3"
$RuleType = "Allow"

Try {
    $ACL = Get-Acl -Path $ACLPath
    $AccessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($Identity,$FileSystemRight,$inheritance,$Propagation,$RuleType)
    $ACL.SetAccessRule($AccessRule)
    $ACL | Set-Acl -Path $ACLPath
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Set folder permissions error: $ErrorMsg"
    Exit 421
}
```
I also have a script in my [GitHub repo](https://github.com/PeterDodemont/Scripts) that uses parameters to pass the required values on the command line. You can find that [here](https://github.com/PeterDodemont/Scripts/blob/main/Misc/Add-FileFolderPermission.ps1).

As always, if you have any questions feel free to reach out.

You can find more details about the class used to set the access rule [here](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesystemaccessrule?view=windowsdesktop-5.0).