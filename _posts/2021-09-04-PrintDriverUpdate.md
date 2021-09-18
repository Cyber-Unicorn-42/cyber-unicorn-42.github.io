---
layout: post
read_time: true
show_date: true
title:  Fixing the point and print driver update prompt
date:   2021-09-04 10:00:00 +1000
description: Fixing the printer driver prompts from users without compromising your security.
img: posts/2021-09-04-PrintDriverUpdate/hero.png
tags: [PrintNightmare,Drivers,Intune,PowerShell,EndpointManager,MEM]
author: Peter Dodemont
---
In this article I will go through what I did to get rid of the driver update prompts that users have been getting after Microsoft's August security patches. The patches included a change in the default behavior of point and print driver installations ([see here](https://msrc-blog.microsoft.com/2021/08/10/point-and-print-default-behavior-change/)). After deployment of the patches non-admin users were no longer able to install drivers using point and print unless some registry tweaks where applied.
This change was causing a lot of our users to get the prompt you see in the hero image at the top of the post. Initially I expected it was a once off due to the change in behavior, so we proceeded to fix it manually for each user that logged a call. Since I work in a SMB and a significant amount of our workforce is unable to go into the offices because of COVID lockdowns the number of requests coming in wasn't too bad. All was good for about a week, then the messages re-appeared, this looked like it was going to need some investigation.
If you just want the scripts I used they are in my [GitHub Scripts repo](https://github.com/PeterDodemont/Scripts/tree/main), and a breakdown of how they work can be found [here](#Automating-The-Driver-Updates).

## The Type 3 driver
I started my investigations in the most logical place, comparing the drivers available on the print server against the locally loaded files. In the process I noticed some of the files in the driver were signed by Microsoft instead of HP. While this seems unusual, I didn't really think anything of it. After comparing a decent amount of files, I did come to the conclusion that some of the files where different versions. This was odd as we hadn't updated the drivers on the server at all, so I was not expecting any differences.
I started doing some research on why some of the driver files might have been different. What I discovered is that when using a Type 3 print driver, some vendors rely on certain files that are provided by Microsoft as part of the OS. Which means that any of those files can be changed when installing Windows Updates, not just on the server, but also on the client. This seemed like the cause for the renewed printer driver update prompt, and sure enough the server had just received its patches as part of our regular patching cycle. This didn't bring me any closer to a solution, but I now had something to go on.

## The Type 4 driver
While trying to find information of how other people were dealing with the problem, I found people mostly just applied the registry tweak to restore the old behavior. Being security conscious, I did not find this solution acceptable in the long term (I did apply it as a short-term work around to stop the flow of calls into our service desk).
In my research on the issue, I came across numerous articles and posts talking about Type 4 printer drivers. I had limited knowledge of the difference between type 3 and 4 printer drivers, so I started looking into it. What I discovered is that with type 3 drivers the driver gets downloaded to the end user device using point and print, but with type 4 a generic Microsoft driver can be loaded and only some configuration files are copied down. Crucially this driver is included on Windows 10 devices out of the box, so doesn't require installation. I had found my solution, simply change the driver on the print server to type 4. During my research I had come across numerous people saying type 4 drivers were not well supported by most vendors, I was in luck though the HP printer model we use across the business has a type 4 driver. I proceeded to download the driver and change it on one of our printers as a test before applying it across the business. I had done driver changes before so I was confident it would be a simple process, oh boy was I wrong.

## The driver type change issue
After I changed the driver on the server I removed and re-added the printer on a test device but upon checking the driver, it was still using the old one. This was very surprising but ok, maybe this device has some issues, so I proceed to manually remove the old driver from the device, re-add the printer and everything is as expected. Testing the printer, everything works as expected. I then proceed to let the service desk know that the driver was updated on one of the printers and that it will update in the next few hours for everyone that has that printer connected.
The next day one of the service desk techs reaches out, and his driver has still not updated. Now people are also complaining that print settings are not being accepted, and an error page is printed after each print. Time to revert the driver back and do some more testing.
"Luckily" some of our offices are empty because of COVID lockdowns, so I have access to an unused printer I can use for testing. After testing lots of different scenario's I find that updating drivers on the print server only updates the driver locally if the driver type is the same, changing from type 3 to type 4 doesn't ever update the driver locally. I do some more testing and find out that not only does the driver never update, but if the client has a type 3 driver in their local driver cache Windows will install the type 3 driver even if the print server is using a type 4 driver. I can only assume this is because the type 3 driver would be considered more compatible than the generic driver used for type 4.

## Automating the driver updates
Knowing what was happening I could now set out to try and solve the issue. The path to take was pretty clear, write a script that will collect all printers using the old type 3 driver, then remove those printers. Next, I'd need to remove the driver followed by finally re-adding the printers which now use the type 4 driver.
You can find the PowerShell script ([Remove-PrintDriver.ps1](https://github.com/PeterDodemont/Scripts/tree/main/Misc/Remove-PrintDriver.ps1)) in my [GitHub Scripts repo](https://github.com/PeterDodemont/Scripts/tree/main) under [Misc](https://github.com/PeterDodemont/Scripts/tree/main/Misc). Below is the breakdown of how it works.

The first part is simply setting some variable that will be used for matching the printer and drivers. As well as listing the NetBIOS and FQDN of the print server(s).
```powershell
$CurrentPrinterDriverName = "*HP Color*"
$DriverStoreInfName= "*hprub32a_x64.inf"
$PrintServerNamesArray=@("prt-01","prt-01.securitypete.com")
```

The 2nd part is where the driver gets collected and put into a variable. You'll notice there is 2 variables for the driver. This is because the driver needs to be removed from the "print server" on the client device as well as from the Windows driver store.
```powershell
$CurrentDriver = Get-PrinterDriver | where {$_.name -like $CurrentPrinterDriverName}
$DriverStoreDriver = Get-WindowsDriver -Online | where {$_.OriginalFileName -like $DriverStoreInfName}
```

I then check if the Temp directory exists, if not it gets created. I then create 2 scripts that will be used in scheduled tasks later.
```powershell
# Create temp directory if it does not exist
If (!(Test-Path $Env:SystemDrive\Temp)){New-Item -Path $Env:SystemDrive\ -Name Temp -ItemType Directory -Force > $null}

# Create base powershell files for use in scheduled tasks later
New-Item -Path $Env:SystemDrive\Temp -Name RemoveUserPrinters.ps1 -Force -ItemType File > $null
New-Item -Path $Env:SystemDrive\Temp -Name ReAddUserPrinters.ps1 -Force -ItemType File > $null

# Create Variables for easy retrieval of the script paths
$RemoveUserPrintersPath = "$Env:SystemDrive\Temp\RemoveUserPrinters.ps1"
$ReAddUserPrintersPath = "$Env:SystemDrive\Temp\ReAddUserPrinters.ps1"
```

The 4th part is the collecting the username and SID of the currently logged in user. I get the username from WMI, as it's in domain\username format I spit it into separate domain and username variables. Finally, I use the separate variables to get the SID.
```powershell
# Get currently logged in user
$CurrentLoggedInUser = (Get-WmiObject -Class Win32_ComputerSystem -Property Username).Username

# Split the username into domain and username
$CurrentUserSplit = $CurrentLoggedInUser.Split("\\")
$CurrentDomain = $CurrentUserSplit[0]
$CurrentUsername = $CurrentUserSplit[1]

# Get the SID of the currently logged in user
$CurrentUserSID = ([wmi]"win32_userAccount.domain='$CurrentDomain',Name='$CurrentUsername'").SID
```

The next section creates a PSDrive so I can more easily access the HKEY_USERS hive.
```powershell
# Create new PSDrive to access HKEY_Users
Try{New-PSDrive -PSProvider Registry -Name HKU -Root HKEY_USERS > $null}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Printer PSDrive Creation Error: $ErrorMsg"
}
```

The following section gets a list of all the network printers currently installed for the user using the PSDrive and SID.
```powershell
# Get all user's network printer entries from the registry
$RegistryPrinters = Get-ChildItem HKU:\$CurrentUserSID\Printers\Connections
```

If any network printers are found I iterate through them and add them to the removal script. And if the print server of the printer matches any the specified print servers, I add it to the script responsible for re-adding the printers.
```powershell
# Run through each registry entry and get the Printer Name and Server to put in the powershell scripts.
If($RegistryPrinters){
    Try {
        ForEach ($RegistryPrinter in $RegistryPrinters){
            $RegistryPrinterPath = $RegistryPrinter.Name
            $RegistryPrinterName = $RegistryPrinter.Name.Split(",")[-1]
            $RegistryPrinterServer = Get-ItemPropertyValue Registry::$RegistryPrinterPath -Name Server
            $RegistryPrinterNetworkPath = $RegistryPrinterServer + "\" + $RegistryPrinterName

            # Create line for printer removal and add it to the powershell scripts
            $PrinterRemovalLine = 'Remove-Printer -Name ' + $RegistryPrinterNetworkPath
            Add-Content -Path $RemoveUserPrintersPath -Value $PrinterRemovalLine
            # Clear the line variable
            Clear-Variable -Name PrinterRemovalLine -Force

            # Check if the servername of the printer matches any of the specified print servers. If it does add a line to the printer re-add script.
            ForEach ($PrintServerName in $PrintServerNamesArray){
                If ($RegistryPrinterServer -like "\\$PrintServerName") {
                    $PrinterReAddLine = 'Add-Printer -ConnectionName ' + $RegistryPrinterNetworkPath
                    Add-Content -Path $ReAddUserPrintersPath -Value $PrinterReAddLine
                    # Clear the line variable
                    Clear-Variable -Name PrinterReAddLine -Force
                }
            }
        }
    }
    Catch {
        $ErrorMsg = $_.Exception.Message
        Write-Host "Printer Script Creation Error: $ErrorMsg"
    }
}
```

Next, I create a scheduled task to run the printer removal as the user. This needs to be done because network printers are user specific and not accessible under the system account. Once created the task gets started and its status get checked every 15 seconds. When it has completed the task gets removed.
```powershell
# Create a Scheduled Task for printer removal. Then run it and wait till it completes by checking if the last printer to be removed still exists. Finally remove the task
If($RegistryPrinters){
    Try {
        $RemovalTaskAction = New-ScheduledTaskAction -Execute "Powershell.exe" -Argument "-NoLogo -NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -Command $RemoveUserPrintersPath"
        $RemovalTrigger = New-ScheduledTaskTrigger -AtLogOn
        $RemovalPrincipal = New-ScheduledTaskPrincipal -UserId $CurrentLoggedInUser
        $RemovalTask = New-ScheduledTask -Action $RemovalTaskAction -Trigger $RemovalTrigger -Principal $RemovalPrincipal
        Register-ScheduledTask -TaskName PrinterRemoval -InputObject $RemovalTask > $null
        Start-ScheduledTask -TaskName PrinterRemoval
        Do {
            Start-sleep -Seconds 15
            $RemovalProgressState = (Get-ScheduledTask -TaskName PrinterRemoval).State
            $RemovalProgressResult = (Get-ScheduledTaskInfo -TaskName PrinterRemoval).LastTaskResult
            If (($RemovalProgressState -eq "Ready") -And ($RemovalProgressResult -ne "0")){
                Unregister-ScheduledTask -TaskName PrinterRemoval -Confirm:$false
                Throw "Printer Removal Scheduled Task Error Code: $RemovalProgressResult"
            }
        }
        While ($RemovalProgressState -ne "Ready")
        Unregister-ScheduledTask -TaskName PrinterRemoval -Confirm:$false

    }
    Catch {
        $ErrorMsg = $_.Exception.Message
        Write-Host "Printer Removal Error: $ErrorMsg"
    }
}
```

After the scheduled task is delete the removal script gets deleted.
```powershell
# Delete the removal script file
Try {Remove-Item -Path $RemoveUserPrintersPath -Force}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Printer Removal Script Seletion Error: $ErrorMsg"
}
```

Then comes the removal of the print driver from the "print server", if the driver specified is found. Sometimes removing the printers does not properly mark the driver as no longer in use, so I get the driver's print processor and environment from WMI. With those I can locate the registry keys of the print processor that the driver uses. Those keys are then renamed before the print spooler is restarted; this clears the driver for deletion. Once deleted I rename the registry keys back to their original value so other printers and drivers using that processor keep working as expected. Finally, I restart the spooler once again.
```powershell
# Remove the current printer driver from the "print server"
If ($CurrentDriver){
    Try {
        # Get driver name
        $CurrentDriverName = $CurrentDriver.Name
        # Get driver print processor
        $CurrentDriverProcessor = (Get-WmiObject -Namespace Root\StandardCimv2 -Class MSFT_PrinterDriver | where {$_.Name -eq $CurrentDriverName}).PrintProcessor
        # Get driver print environment
        $CurrentDriverEnvironment = (Get-WmiObject -Namespace Root\StandardCimv2 -Class MSFT_PrinterDriver | where {$_.Name -eq $CurrentDriverName}).PrinterEnvironment
        # Generate names for print processor regkey
        $PrintProcessorRegKeyCurrentName = $CurrentDriverProcessor
        $PrintProcessorRegKeyNewName = $PrintProcessorRegKeyCurrentName + ".old"
        # Set paths for print processor regkey
        $PrintProcessorRegKeyCurrentPath = "HKLM:\SYSTEM\CurrentControlSet\Control\Print\Environments\" + $CurrentDriverEnvironment + "\Print Processors\" + $PrintProcessorRegKeyCurrentName
        $PrintProcessorRegKeyRenamedPath = "HKLM:\SYSTEM\CurrentControlSet\Control\Print\Environments\" + $CurrentDriverEnvironment + "\Print Processors\" + $PrintProcessorRegKeyNewName
        # rename print processor to Remove driver from use
        Rename-Item -Path $PrintProcessorRegKeyCurrentPath -NewName $PrintProcessorRegKeyNewName -Force
        # Restart Print Spooler Service
        Restart-Service -Name spooler
        #Remove Printer driver
        Remove-PrinterDriver -Name $CurrentDriverName -ErrorAction Stop
        # rename print processor back to original name
        Rename-Item -Path $PrintProcessorRegKeyRenamedPath -NewName $PrintProcessorRegKeyCurrentName -Force
        # Restart Print Spooler Service
        Restart-Service -Name spooler
    }
    Catch {
        $ErrorMsg = $_.Exception.Message
        Write-Host "Driver Removal Error: $ErrorMsg"
    }
}
```

After the removing the driver form the "print server" it needs to be removed from the Windows driver store. There is no built-in PowerShell cmdlet for this, so I use pnputil for the removal. The removal only runs if the specified driver was found in the Windows driver store.
```powershell
# Delete driver from windows driver store
If ($DriverStoreDriver) {
    $PnpUtilRun = pnputil /delete-driver $DriverStoreDriver.Driver 2>&1
    If ($LASTEXITCODE -ne 0){
        Throw "Printer PnpUtil Error: $PnpUtilRun"
    } 
}
```

As with the removal of the printers, the re-adding of the printers needs to be done under the user's context. I create another scheduled task same as before but now for re-adding the printers. It also keeps track of its progress and once it has run it task gets deleted.
```powershell
# Create a Scheduled Task for Re-adding the printer. Then run it and wait till it completes by checking if the last printer to be re-add exists. Finally remove the task.
If($RegistryPrinters){
    Try {
        $ReAddTaskAction = New-ScheduledTaskAction -Execute "Powershell.exe" -Argument "-NoLogo -NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -Command $ReAddUserPrintersPath"
        $ReAddTrigger = New-ScheduledTaskTrigger -AtLogOn
        $ReAddPrincipal = New-ScheduledTaskPrincipal -UserId $CurrentLoggedInUser
        $ReAddTask = New-ScheduledTask -Action $ReAddTaskAction -Trigger $ReAddTrigger -Principal $ReAddPrincipal
        Register-ScheduledTask -TaskName PrinterReAdd -InputObject $ReAddTask > $null
        Start-ScheduledTask -TaskName PrinterReAdd
        Do {
            Start-sleep -Seconds 15
            $ReAddProgressState = (Get-ScheduledTask -TaskName PrinterReAdd).State
            $ReAddProgressResult = (Get-ScheduledTaskInfo -TaskName PrinterReAdd).LastTaskResult
            If (($ReAddProgressState -eq "Ready") -And ($ReAddProgressResult -ne "0")){
                Unregister-ScheduledTask -TaskName PrinterReAdd -Confirm:$false
                Throw "Printer Re-Adding Scheduled Task Error Code: $ReAddProgressResult"
            }
        }
        While ($ReAddProgressState -ne "Ready")
        Unregister-ScheduledTask -TaskName PrinterReAdd -Confirm:$false

    }
    Catch {
        $ErrorMsg = $_.Exception.Message
        Write-Host "Printer Re-Add Error: $ErrorMsg"
    }
}
```

Finally, I delete the script responsible for re-adding the printers as well as the PSDrive I created.
```powershell
# Delete the re-adding script file
Try {Remove-Item -Path $ReAddUserPrintersPath -Force}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Printer Re-Add Script Deletion Error: $ErrorMsg"
}

# Remove PSDrive
Try{Remove-PSDrive -Name HKU -Force}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Printer PSDrive Removal Error: $ErrorMsg"
}
```

## Deployment using Intune Proactive Remediations
If you are not familiar with Intune Proactive Remediations have a look [here](https://docs.microsoft.com/en-us/mem/analytics/proactive-remediations). Basically, it allows you to specify a state you want to achieve and if that state isn't met it will run a script to remediate.
To leverage this feature, I wrote a little script that will check if a particular driver is installed and if it is, the remediation will kick in. This script([ProactiveRem-Driver-Detection.ps1](https://github.com/PeterDodemont/Scripts/tree/main/Intune/ProactiveRem-Driver-Detection.ps1)) can also be found in my [GitHub Scripts repo](https://github.com/PeterDodemont/Scripts/tree/main) under [Intune](https://github.com/PeterDodemont/Scripts/tree/main/Intune).
The script is fairly simple. You set a variable for the driver matching and it provides the required exit code depending on if the driver was found or not.
```powershell
$DriverStoreInfName= "*hprub32a_x64.inf"

Try {
    If (!(Get-WindowsDriver -Online | where {$_.OriginalFileName -like $DriverStoreInfName} -ErrorAction Stop )){
        Write-host "Driver Not Found"
        #Exit 0
    }
    Else{
        Write-Host "Driver Found"
        #Exit 1
    }
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-host "Error $ErrorMsg"
    #Exit 1
}
```

## Type 4 printer limitations
Before deciding to implement this, you should be aware there are some limitations with type 4 drivers. As they rely on a generic driver a lot of additional functionality from the printers is not available unless the vendor provides an application that can be run on each client device to access this additional functionality. So be sure to test what functionality will not be available without this additional app.

As always if you have any questions, please reach out.