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
There is plenty of information ot there on how to set permissions via powershell, but most of it does not seem to include much information on how to set propagation and inheritance. So I will focus on those two items in this article. I will cover off how to set the permissions as well, so you can just have a single source to refer to should you need it.

## Propagation and Inheritance
Before getting into how to set the permissions I want to give a quick refresher on what these 2 properties are, what they do and how they are linked.
Let's start with inheritance, this is a property on the file or folder that will indicate if this file or folder will have the same permissions as it's parent. Inheritance is recursive so unless broken somewhere along the line all the permissions from any parent folder all the way up to the root will be applied. You can find inheritance in the advanced security properties of a file or folder in the GUI.
![Inheritance in the GUI](/assets/img/posts/2021-12-04-powershell-permissions/inheritance-gui.png "Inheritance in the GUI")
Propagation indicates if that particular access rule is allowed to be passed on the any child items, and which item types it can be passed onto to. Access rules are just the individual entries that make up the access control list (or ACL). They are also referred to as access control entries (or ACE), but I PowerShell refers to them as access rules so that is what I'll be using. Propagation is only available on folders (since files do not have any child items). There are a few different options for propagation:
* This folder : Indicates that the access rule can only apply to the folder it has been set on
* Subfolders : Indicates that the access rule can be set on any subfolder
* Files : Indicates that the access rule can be set on any files
These can be combined to control where the access rule will get applied to. The most common is "This folder, subfolders and files" as that allows it to be set on any child item. But it is not uncommon to use just "This folder" if you want a particular permission on a single folder. "Files" is also used frequently if you want to prevent users from changing the folder structure but allowing them to add, remove or modify files.
The propagation details of the access rule can also be found on the advanced properties windows of the item in the GUI.
![Propagation in the GUI](/assets/img/posts/2021-12-04-powershell-permissions/propagation-gui.png "Propagation in the GUI")
These 2 properties are closely linked. E.g. if you disable inheritance, no access rule will come across from a parent item no matter what the propagation of the access rule might be. And in the same way, if you set propagation to not go beyond this folder, it doesn't matter what you do with inheritance of any subfolders they will not get that particular access rule.
This means that getting these right when setting permissions via PowerShell is just as important as setting the actual permissions. Even more so because the defaults in PowerShell are different from the defaults the GUI uses. The GUI defaults to full propagated (i.e. this folder, subfolders and files) and inheritance enabled. PowerShell is the exact opposite, it disables inheritance and sets the propagation to be for a this folder only.

## Setting The ACL
Setting the ACL is not overly complicated. It is only an handful of lines.
First, I grab the existing ACL from the item.
```powershell
$ACL = Get-Acl -Path C:\temp
```
Next, I create a new access rule to be added to the ACL. This is where the propagation and inheritance comes in. As you can see from the line below there are a number of properties that you need to define when setting the access rule. The properties need to be in a particular order and some are optional. In order they are:
* The name or SID of the user or group the access rule will apply to.
* The particular permission the access rule will apply. See [here](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesystemrights?view=windowsdesktop-5.0) for a list of possible values.
* The inheritance of the access rule. This is an optional property.
* The propagation of the access rule. This is also an optional property.
* The type of access rule. This is either Allow or Deny.
You'll notice that the propagation and inheritance properties are numeric values. You can find information on which values to use [here for propagation](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.propagationflags?view=windowsdesktop-5.0) and [here for inheritance](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.inheritanceflags?view=windowsdesktop-5.0). The values you will find on those pages need to be added together if you want to provide the settings. E.g. if you want to set inheritance to be just for subfolders you set the inheritance property to 1. If you want have inheritance for subfolders and files you set the property to 3 (as I have in my example below).
Note that if you do not supply a value for both propagation and inheritance they will use 0. This means there is no propagation or inheritance for that access rule.
```powershell
$AccessRule = New-Object System.Security.AccessControl.FileSystemAccessRule("Users","Modify","3","0","Allow")
```
Then, you add the access rule to the set of access rules already in the ACL
```powershell
$ACL.SetAccessRule($AccessRule)
```
And finally you write the ACL back onto the file or folder.
```powershell
$ACL | Set-Acl -Path c:\Temp
```
All together this will look like the below. As with all my other scripts it's wrapped in a Try-Catch block for error handling. I also generally use variables or parameters so I can easily re-use the script in the future.
```powershell
$ACLPath = "C:\Temp"

Try {
    $ACL = Get-Acl -Path $ACLPath
    $AccessRule = New-Object System.Security.AccessControl.FileSystemAccessRule("Users","Modify","3","0","Allow")
    $ACL.SetAccessRule($AccessRule)
    $ACL | Set-Acl -Path $ACLPath
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Set folder permissions error: $ErrorMsg"
    Exit 421
}
```

As always, if you have any questions feel free to reach out.

You can find more details about the class used to set the access rule [here](https://docs.microsoft.com/en-us/dotnet/api/system.security.accesscontrol.filesystemaccessrule?view=windowsdesktop-5.0).