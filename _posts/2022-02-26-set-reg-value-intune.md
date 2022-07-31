---
layout: post
read_time: true
show_date: true
title:  Setting and checking registry values with MEM proactive remediations
date:   2022-02-26 10:00:00 +1100
description: How to set and check registry values for both the current user and the local machine using Microsoft Endpoint Manager proactive remediations.
img: posts/2022-02-26-set-reg-value-intune/hero.png
tags: [intune/endpoint-manager,powershell]
author: Peter Dodemont
---
MEM proactive remediations are a great way to replicate what you can do in an on-prem environment with group policy preferences. I personally even prefer applying settings through MEM instead of group policy these days, as anything configured in MEM will apply even if the device is not currently connected to the corporate network (either by being in an office or via a VPN).
Setting the value of registry items is one of those group policy preferences, so let's have a look at how we can do this via a MEM proactive remediation.

I have created a single set of scripts that is able to be used for registry values either in the current user hive (HKCU) or in the local machine hive (HKLM) of the registry depending on your needs.

MEM proactive remediation requires 2 scripts, 1 to detect whatever it is to change and 1 to apply the changes. Both scripts can be found in my [GitHub Repo](https://github.com/PeterDodemont/Scripts). The detection script is [here](https://github.com/PeterDodemont/Scripts/blob/main/Intune/ProactiveRem-RegValue-Detection.ps1) and the script to set the keys is [here](https://github.com/PeterDodemont/Scripts/blob/main/Misc/Set-RegValue.ps1).
Let's look at the detection script first.

## The Detection Script
As proactive remediations require the script to be loaded in full and it is not possible to pass parameters on the command line, the first part is setting variables for all the values that might change. In this case this is the full path to the registry keys including the name of keys, the value you expect the keys to have (all the keys need to have the same value) and if the script will be run under the system account but needs to check values under the currently logged in user.
```powershell
# Set Variables
$RegKeyFullPaths = @("HKCU:\SOFTWARE\Policies\Microsoft\Edge\Recommended\NewTabPageLocation")
$RegKeyExpectedValue = "https://peterdodemont.com"
$CurrentUserAsSystem = $false
```

Next, I check if the $CurrentUserAsSystem variable was set to $true. If it is, I get the details of the currently logged in user from WMI. WMI stores this in the "domain\username" format, but I need them separately, so I split the information from WMI at the "\\" and store each part in a variable. I can then use that information to get the SID of the user account. I need the SID to be able to check the value of the registry key for the current user. When running as the system account, HKCU will be the registry keys and values for the system account. In the registry a user's keys and values are stored under a key in the HKEY_Users hive using the user's SID (since the SID will never change even if other details of the account are changed).
Once I have the SID, I remove any current HKCU mapping in PowerShell and remap it to the section of the registry I need it to point to.  
One thing to note here is that getting the SID in this way only works if there is a local active directory, for devices that are only joined to Azure this will not work (thanks to [@MJGAIL](https://twitter.com/MJGAIL) for helping me discover this).
```powershell
# Check if you need to check it for the current user as system
If ($CurrentUserAsSystem -eq $true){
    # Get currently logged in user
    $CurrentLoggedInUser = (Get-WmiObject -Class Win32_ComputerSystem -Property Username).Username

    # Split the username into domain and username
    $CurrentUserSplit = $CurrentLoggedInUser.Split("\\")
    $CurrentDomain = $CurrentUserSplit[0]
    $CurrentUsername = $CurrentUserSplit[1]

    # Get the SID of the currently logged in user
    $CurrentUserSID = ([wmi]"win32_userAccount.domain='$CurrentDomain',Name='$CurrentUsername'").SID

    # Remove Current PSDrive pointing to HKCU
    Remove-PSDrive HKCU

    # Create new PSDrive for HKCU pointing to the SID of the currently logged in user under HKEY_USERS
    New-PSDrive -PSProvider Registry -Name HKCU -Root HKEY_USERS\$CurrentUserSID > $null
}
```
Then, for each path provided, I split it into the path and the name of the registry key. I could do this in a similar way to what I did for the user details above, but there is a built-in PowerShell function that will do this for me as well.
Now that I have both the path and name, I get the value of the key and store that in a variable I can use to check against.
* If the value matches what was specified at the start of the script, I output a message showing which key matches and proceed to the next path provided.
* If the key doesn't match, I output a message saying which key didn't match. I then check once again if the script is being run as the system account but checking values for the currently logged in user. If it is, I remove the HKCU drive and remap it back to the original location in the registry.

Once all paths have been checked and there have been no errors, I remap the HKCU drive to the original location if it has been remapped.

The section that does the checking is also wrapped in a Try-Catch block for error handling.
```powershell
# Run check through each registry path
ForEach ($RegKeyFullPath in $RegKeyFullPaths) {
    
    # Get the parent and the leaf from each path
    $RegKeyPath = Split-Path $RegKeyFullPath -Parent
    $RegKey = Split-Path $RegKeyFullPath -Leaf

    # Get registry key value
    $RegKeyCurrentValue = Get-ItemPropertyValue -Path $RegKeyPath -Name $RegKey -ErrorAction SilentlyContinue

    # Check registry key value. If it doesn't match trigger remediation.
    Try {
        If ($RegKeyCurrentValue -eq $RegKeyExpectedValue){
            Write-host "Registry key $RegKeyPath\$RegKey has correct value."
        }
        Else{
            Write-Host "Registry key $RegKeyPath\$RegKey has incorrect value."
            # Check if you need to check it for the current user as system
            If ($CurrentUserAsSystem -eq $true){
                # Restore original PSDrive
                Remove-PSDrive HKCU
                New-PSDrive -PSProvider Registry -Name HKCU -Root HKEY_CURRENT_USER > $null
            }
            Exit 1
        }
    }
    Catch {
        $ErrorMsg = $_.Exception.Message
        Write-host "Error $ErrorMsg"
        # Check if you need to check it for the current user as system
        If ($CurrentUserAsSystem -eq $true){
            # Restore original PSDrive
            Remove-PSDrive HKCU
            New-PSDrive -PSProvider Registry -Name HKCU -Root HKEY_CURRENT_USER > $null
        }
        Exit 1
    }
}

# Check if you need to check it for the current user as system
If ($CurrentUserAsSystem -eq $true){
    # Restore original PSDrive
    Remove-PSDrive HKCU
    New-PSDrive -PSProvider Registry -Name HKCU -Root HKEY_CURRENT_USER > $null
}
Exit 0
```

## The Remediation Script
Just like the detection script, the remediation script starts with setting variables for the different values. There are 3 variables that are identical to detection script, but I added a 4th variable that is used to set the type of registry key being created. There is only a limited number of valid registry types, and I included a link to where you can find the correct values to use.
```powershell
# Set Variables
$RegKeyFullPaths = @("HKCU:\SOFTWARE\Policies\Microsoft\Edge\Recommended\NewTabPageLocation")
$RegKeyValue = "https://peterdodemont.com"
$RegType = "String" #See https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/set-itemproperty for support types
$CurrentUserAsSystem = $false  # Use either $true or $false
```

The following section is identical to the detection script so I won't cover it again, but I will leave to code below for completeness.
```powershell
# Check if you need to check it for the current user as system
If ($CurrentUserAsSystem -eq $true){
    # Get currently logged in user
    $CurrentLoggedInUser = (Get-WmiObject -Class Win32_ComputerSystem -Property Username).Username

    # Split the username into domain and username
    $CurrentUserSplit = $CurrentLoggedInUser.Split("\\")
    $CurrentDomain = $CurrentUserSplit[0]
    $CurrentUsername = $CurrentUserSplit[1]

    # Get the SID of the currently logged in user
    $CurrentUserSID = ([wmi]"win32_userAccount.domain='$CurrentDomain',Name='$CurrentUsername'").SID

    # Remove Current PSDrive pointing to HKCU
    Remove-PSDrive HKCU

    # Create new PSDrive for HKCU pointing to the SID of the currently logged in user under HKEY_USERS
    New-PSDrive -PSProvider Registry -Name HKCU -Root HKEY_USERS\$CurrentUserSID > $null
}
```

The final part of the script runs through each registry key provided and runs the following steps on each:
* Split the path provided into the path and the name of the key
* Check if the path already exists or not, if it doesn't it will create the path
* Set or create the registry key of the correct type with the provided value.
As before, the creation of the path and setting/creating of the registry key are wrapped in Try-Catch blocks for error handling. And the HKCU drive will also be remapped before the scripts exits whether this happens from an error or before successful completion of the script.
```powershell
# Run through each registry path
ForEach ($RegKeyFullPath in $RegKeyFullPaths) {
    
    # Get the parent and the leaf from each path
    $RegKeyPath = Split-Path $RegKeyFullPath -Parent
    $RegKey = Split-Path $RegKeyFullPath -Leaf

    # Check if the registry path exists if not create it
    If (!(Test-Path $RegKeyPath)){
        Try {
            # Create the new path
            New-Item $RegKeyPath -ErrorAction Stop -Force > $null
            Write-Host "Registry path created successfully"
        }
        Catch {
            $ErrorMsg = $_.Exception.Message
            Write-host "Error creating registry path: $ErrorMsg"
            If ($CurrentUserAsSystem -eq $true){
                # Restore original PSDrive
                Remove-PSDrive HKCU
                New-PSDrive -PSProvider Registry -Name HKCU -Root HKEY_CURRENT_USER > $null
            }
            Exit 1
        }
    }
    # Set value of registry key.
    Try {
        Set-ItemProperty -Path $RegKeyPath -Name $RegKey -Value $RegKeyValue -Type $RegType -ErrorAction Stop -Force
        Write-Host "Registry value set correctly"
        If ($CurrentUserAsSystem -eq $true){
            # Restore original PSDrive
            Remove-PSDrive HKCU
            New-PSDrive -PSProvider Registry -Name HKCU -Root HKEY_CURRENT_USER > $null
        }
    }
    Catch {
        $ErrorMsg = $_.Exception.Message
        Write-host "Error setting the registry value: $ErrorMsg"
        If ($CurrentUserAsSystem -eq $true){
            # Restore original PSDrive
            Remove-PSDrive HKCU
            New-PSDrive -PSProvider Registry -Name HKCU -Root HKEY_CURRENT_USER > $null
        }
        Exit 1
    }
}
```

That is all there is too it. I hope this has been useful to you, and as always if you have any questions or comments please reach out.