---
layout: post
read_time: true
show_date: true
title:  Mapping network printer via Intune/MEM
date:   2021-09-25 10:00:00 +1000
description: How to provide users the ability to map printers in MEM/Intune and automatically map printers based on their location.
img: posts/2021-09-25-map-printers-intune/hero.png
tags: [Printer,Intune,PowerShell,EndpointManager,MEM]
author: Peter Dodemont
---
Adding printers inevitably forms part of the onboarding process for all new users. You can automate this part easily enough through GPO. But what if you also want to provide users with a way to easily map printers through a familiar interface? In this article I'll show you how I did this in MEM. I wrote the scripts for MEM, but they should be able to be used in any deployment product that supports PowerShell. The process is simple, first create some dynamic groups in Azure AD, then create applications in MEM and finally assign the applications to the groups.
If you just want the scripts, the install script is [here](https://github.com/PeterDodemont/Scripts/tree/main/Install-Scripts/Map-Printers.ps1), the detection script is [here](https://github.com/PeterDodemont/Scripts/tree/main/Intune/Printer-Detection.ps1) and the explanation of how it works is [here](#TheScripts).

## The Dynamic Groups
The rule I used for the dynamic groups is very simple
```
(user.physicalDeliveryOfficeName -eq "Sydney")
```
I can keep it this simple because to groups will only be used for deployments in Intune. Even if there are additional accounts in there (e.g. service accounts), it will be of no consequence as these accounts don't have a license to access MEM.

## <a name=TheScripts></a>The Scripts
I created 2 scripts, one for the mapping of the network printers and another for the detection.
### The Mapping Script
The script can be found [here](https://github.com/PeterDodemont/Scripts/blob/main/Install-Scripts/Map-Printers.ps1).
Since I will be re-using this script for each office, I made the script to read the required information from the command line. This way there is no need to modify the script and then create a new .intunewin file for each site.

First, I configure the parameters that need to be provided. I also perform a very basic validation on one of the parameters to try and confirm a FQDN was provided and not a NetBIOS name. I know this validation can probably be more advanced, but it serves its purpose well enough for me. Explanation on the use of each parameter can be found in the script, although the names are pretty self-explanatory.
```powershell
Param
(
[Parameter(Mandatory=$true)]
[ValidateScript({If($_ -like "*.*"){$true}Else{Throw "$_ is not an FQDN. Please enter a FQDN."}})]
[string]
$PrintServerFQDN
,
[Parameter(Mandatory=$true)]
[string[]]
$PrinterShareNames=@()
,
[Parameter(Mandatory=$False)]
[switch]
$RemoveFirst
,
[Parameter(Mandatory=$False)]
[switch]
$RemoveOnly
)
```

Next, I check if the FQDN was provided just as a server name or as a path. If it is just a server name, I adjust the variable to be the path as that is what I need throughout the script. I also get the NetBIOS name of the server since I also use that throughout the script. There is no need to normalize the NetBIOS name as I am using the FQDN that was already normalized.
```powershell
# normalize the Print server FQDN
If ($PrintServerFQDN -notlike "\\*") {$PrintServerFQDN = "\\" + $PrintServerFQDN}

# Get the non FQDN name of the print server from the FQDN
$PrintServerName = $PrintServerFQDN.Split(".")[0]
```

This section first checks if the parameters requesting removal have been passed, then it loops through each of the printers provided and finally removes them if found. It checks both the FQDN and NetBIOS names of the server against each printer. In the existence check I specified to silently continue on errors as an error is thrown when the printer is not found. I'm also not concerned with catching errors for the existence check, but I do want to catch and log removal errors.
```powershell
# Remove printers
If (($RemoveFirst -eq $true) -Or ($RemoveOnly -eq $true)){
    Try {
        Foreach ($Printer in $PrinterShareNames){
            # Generate printer names
            $PrinterNameFQDN = $PrintServerFQDN + "\" + $Printer
            $PrinterName = $PrintServerName + "\" + $Printer

            # Remove Printer if it exists
            If (get-printer -Name $PrinterNameFQDN -ErrorAction SilentlyContinue) {Remove-Printer -Name $PrinterNameFQDN}
            If (get-printer -Name $PrinterName -ErrorAction SilentlyContinue) {Remove-Printer -Name $PrinterName}
        }
    }
    Catch {
        $ErrorMsg = $_.Exception.Message
        Write-Host "Printer removal error: $ErrorMsg"
    }
}
```

The final section (re)adds the printers specified using the FQDN, as that is more reliable than using NetBIOS names. This section only executes if the parameter to only remove the printers was not specified.
```powershell
# (Re)Add printer
If ($RemoveOnly -eq $false){
    Try {
        Foreach ($Printer in $PrinterShareNames){
            # Generate correct printer name for (re)adding
            $PrinterNameFQDN = $PrintServerFQDN + "\" + $Printer

            # (Re)Add printer
            Add-Printer -ConnectionName $PrinterNameFQDN
        }
    }
    Catch {
        $ErrorMsg = $_.Exception.Message
        Write-Host "Printer add error: $ErrorMsg"
    }
}
```
### The Detection Script
The script can be found [here](https://github.com/PeterDodemont/Scripts/blob/main/Intune/Printer-Detection.ps1).
In addition to mapping the printers I also need to be able to detect if the printers were added successfully. Since detection scripts in MEM get loaded directly and not via a command, I need to specify the printers directly in the script. But as they don't need to be in the .intunewin format it's not that big of an issue.

First, I set the variable that contains the printers to be detected using the FQDN.
```powershell
# Set variable with printer names
$Printers = @("\\prt-01.securitypete.com\PRT-SYD-01","\\prt-01.securitypete.com\PRT-SYD-02")
```

Then for each printer provided I check if it exists. If it doesn't an error is written out and the scripts exit with an error code of 1. If it finds the printer it loops through until there are no more printers after which it write out a message to the console and exits with an error code of 0. If any errors happen the script captures the error message and outputs it to the console, it then exits with an error code of 1. All these outputs can be seen in the IntuneManagementExtension.log log file which makes troubleshooting a lot easier.
```powershell
# Check if printers are installed
Try{
    Foreach($Printer in $Printers){
        # Throw error is printer doesn't exist
        If (!(Get-Printer -Name $Printer -ErrorAction SilentlyContinue)){
            Write-Host "$Printer not found"
            Exit 1
        }
    }
    # If no errors exit with success message and exit code
    Write-Host "All printers detected"
    Exit 0
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Printer detection error: $ErrorMsg"
    Exit 1
}
```

## The Application
As this is purely a PowerShell script you will need to package it up into a .intunewin file using the [Microsoft-Win32-Content-Prep-Tool](https://github.com/Microsoft/Microsoft-Win32-Content-Prep-Tool). Since network printers are loaded in the user profile you need to set the "Install behavior" in MEM to "User". You use the "Install command" and "Uninstall command" fields to provide the parameters for the mapping script. Below is a breakdown of the commands I use.
Install command
```
Powershell.exe -NoLogo -NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -Command .\Map-Printers.ps1 -PrintServerFQDN prt-01.securitypete.com -PrinterShareNames PRT-SYD-01,PRT-SYD-02 -RemoveFirst
```
Uninstall command
```
Powershell.exe -NoLogo -NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -Command .\Map-Printers.ps1 -PrintServerFQDN prt-01.securitypete.com -PrinterShareNames PRT-SYD-01,PRT-SYD-02 -RemoveOnly
```

Since it's a PowerShell script it needs to be loaded using the PowerShell executable. I use -NoLogo and -NoProfile to speed up the loading, -ExecutionPolicy Bypass is loaded so the script doesn't need to be signed and finally -WindowStyle Hidden is to hide the PowerShell window from the user. Take note that the PowerShell screen will flash up briefly before being hidden. An explanation on why that is and some workarounds can be found [here](https://github.com/PowerShell/PowerShell/issues/3028).
I use the -Command option instead of the -File option because there are issues with passing a variable as an array using -File.
If you want some more information on the command line options available to powershell.exe, see [here](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_powershell_exe?view=powershell-5.1).
I pass the name of the script to PowerShell as .\Map-Printers.ps1 since that is the PowerShell command for running that script as you type it in a PowerShell console window.
I then proceed to passing all the parameters of the script -PrintServerFQDN followed by the FQDN of the print server, -PrinterShareNames followed by the names of the printers separated by a comma and finally either -RemoveFirst or -RemoveOnly depending on my requirements.
You will also need to specify the detection rules for the application. For that just upload the detection script you modified accordingly.
Finally, you need to assign the application. I usually assign it as required to the relevant dynamic group and make it available to all users. This way any user can install any printer for any office through the familiar Company Portal interface. This also removes the need to create documentation for adding printers.

As always if you have any questions please reach out.