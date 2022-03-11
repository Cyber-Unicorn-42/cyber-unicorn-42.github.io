---
layout: post
read_time: true
show_date: true
title:  Creating and detection files or folders using proactive remediations in MEM
date:   2022-03-11 10:00:00 +1000
description: A little script I wrote to create and detect files or folders using proactive remediation in MEM.
img: posts/2022-03-12-file-folder-creation/hero.png
tags: [intune/endpoint-manager,powershell]
author: Peter Dodemont
---
A little while ago I got a report one of the scripts deployed through MEM was no longer working. Upon investigating it was because the temp folder was missing. The person responsible for the script assumed the existence of a folder (C:\Temp) rather than checking it and creating as needed. As I was unsure if there were more scripts or installs that made the same assumption, I decided to create a proactive remediation I can deploy to all users. It checks if the folder exists already and if it doesn't it will be created.

You can find both scripts I use in my [GitHub Repo](https://github.com/PeterDodemont/Scripts). The detection script is [here](https://github.com/PeterDodemont/Scripts/blob/main/Intune/ProactiveRem-Path-Detection.ps1) and the remediation script [here](https://github.com/PeterDodemont/Scripts/blob/main/Misc/Create-FileAndFolder.ps1).

## The Detection
Detecting the existence of a folder is pretty easy. It only requires a single command, but as with most of my scripts I try to make it so I can easily re-use it easily. I also add in error handling, as even the simplest things can go wrong.

I start of by setting a variable with the different paths to either files or folders that I want to check.

Then I use the Test-Path command, to check if the file or folder exists. This command will return "true" or "false" depending in if the path exists or not. I store the result of the Test-Path command in a variable.

Next, I check that variable and if it is not true, I write out a message saying the path can't be found and exit the script with the error code of 1 which will trigger remediation.

I then repeat this for every path provided. If all the paths are found, I write out a message saying they were all found and exit with an error code of 0.

```powershell
# Set Variables
$Paths = @("$Env:SystemDrive\Temp\Test\path1","$Env:SystemDrive\Test\path1\Path2")

# Check if paths exist, if not trigger remediation.
Try {
    ForEach ($Path in $Paths) {
        $PathTest = Test-Path $Path
        If (!($PathTest)){
            Write-host "$Path Not Found"
            Exit 1
        }
    }
    Write-host "All Paths Found"
    Exit 0
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-host "Error $ErrorMsg"
    Exit 1
}
```

## The Remediation
Creating files and folders is also pretty straight forward in PowerShell. You can use the New-Item command for both, all that is needed is to replace the Itemtype property accordingly. Because both the folder and file section of the script are virtually identical other than the aforementioned Itemtype property and some variable names, I will only go over the folder section.

As with the detection script I first define variables for the files and folders I want to create.

Then, I check if any the variable for the folder exists and contains data. The folder creation part of the script will only run if the variable exists and contains data.

Once I know there is data in the variable, I first check if the folder already exists. I do this, so I can specify multiple folders in the same proactive remediation and don't get errors when 1 of the folders already exists but not the others. The check is done in the exact same way as I do for the detection script, so I won't go over it again.

After testing for the existence of the folder, I spit the provided path into the parent and the leaf. I need to specify each separately later in the script, so both are store in a variable. The built-in Split-Path command in PowerShell splits paths into their parent and leaf, and it works across both files and folders as well.

Finally, I check the variable where the folder existence result was stored and if the folder is not found, I create it using the New-Item command and providing the parent and leaf using the variables created earlier. I also specify the Itemtype as "Directory" to make sure a folder is created (if you want to create a file you specify the Itemtype property as "File").

This process gets repeated for each folder specified, before continuing on to do the same process fo the files.
As usual everything is wrapped in try-catch blocks to detect and report any error through to the MEM console.

```powershell
# Set Variables
$FolderPaths = @("$Env:SystemDrive\Temp\Test\path1","$Env:SystemDrive\Test\path1\Path2")
$FilePaths = @("$Env:SystemDrive\Temp\Test.txt","$Env:SystemDrive\Test\path1\Path2\test.txt")

# Check if folders exist, if not create them.
If ($FolderPaths){
    Try {
        ForEach ($FolderPath in $FolderPaths) {
            $FolderTest = Test-Path $FolderPath

            # Get the parent and the leaf from each path
            $FolderParent = Split-Path $FolderPath -Parent
            $FolderLeaf = Split-Path $FolderPath -Leaf

            If (!($FolderTest)){
                New-item -Path $FolderParent -Name $FolderLeaf -Force -ItemType Directory
                Write-Host "Folder $FolderPath Created Successfully."
            }
            Else {
                Write-Host "Folder $FolderPath exists already."
            }
        }
        Write-Host "All Folders Created Successfully."
    }
    Catch {
        $ErrorMsg = $_.Exception.Message
        Write-host "Error $ErrorMsg"
        Exit 1
    }
}
```

That is all there is to using proactive remediations to detect and create one or multiple files and/or folders. As always, if you have any questions or comments please reach out.