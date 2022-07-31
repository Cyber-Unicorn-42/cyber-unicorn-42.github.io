---
layout: post
read_time: true
show_date: true
title:  Replacing Strings In A Flash
date:   2022-06-11 10:00:00 +1000
description: Replacing strings in files using PowerShell
img: posts/2022-06-11-replace-strings-powershell/hero.jpg
tags: [powershell]
author: Peter Dodemont
---
I have been in many situations where I have had to go through a config file and update a particular string in the file with a new one. Most of the time this happens when someone uses IP addresses to configure something instead of using a DNS name. Some admins will say that they prefer to use IP addresses because that means everything keeps working even if there are problems with name resolution. My answer to this is that if you are having name resolution problems, you probably have bigger problems than this one application not working :)  
So today I will go through a script I wrote recently when I had to replace a config entry in several files spread through multiple directories.  
The script is very similar to the one I use when I want to change config files and restart a specific service afterward (which you can find [here](\replace-and-restart.html)). It does differ in that for this particular use case I did not need to restart any service (so I did not include that in the script although it can easily be copied over from the other script), and I had a large number of files that needed to be updated and all files had the same name but were in different folders.

## The Script
The script is in my GitHub repo [here](https://github.com/PeterDodemont/Scripts/tree/main/Misc).
As usual, the script starts with several parameters. There is the transcript path I always include for troubleshooting purposes, then we have some parameters for the file path to search, the filename to search for, if the search should happen recursively into sub folders as well, and of course the new and old strings.
```powershell
Param
(
[Parameter(Mandatory=$false)]
[String]
$TranscriptPath
,
[Parameter(Mandatory=$true)]
[string]
$FilePath
,
[Parameter(Mandatory=$true)]
[string]
$FileName
,
[Parameter(Mandatory=$false)]
[switch]
$Recurse
,
[Parameter(Mandatory=$true)]
[string]
$OldString
,
[Parameter(Mandatory=$true)]
[string]
$NewString
)
```

The first part of the script starts the transcript if I am passing a path via the "TranscriptPath" parameter. This is useful when running scripts in non-interactive or hidden windows. Or if I am having someone else run the script, they can then just send me the log file created to show the results.
```powershell
# Start transcript when Transcript parameter is passed.
Try {
    If ($TranscriptPath){
        Start-Transcript -Path "$TranscriptPath\ReplaceString.log" -Force
    }
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Unable to start transcript: $ErrorMsg"
    Exit 431
}
```

The next part is the first section which deals with the actual purpose of the script. In this part, I create a variable called "FileLocations" that will store the path for any file with the provided filename that is located in the folder or sub-folders (if the recurse parameter is specified) provided. I do this by calling Get-ChildItem with the path and name parameters. 
```powershell
# Get all the files with the specified filenames in the specified location
Try {
    If ($Recurse -eq $true) {
        $FileLocations = Get-ChildItem -Path $FilePath -Name $FileName -Recurse
    }
    Else {
        $FileLocations = Get-ChildItem -Path $FilePath -Name $FileName
    }
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Error Finding Files: $ErrorMsg"
    Exit 432
}
```

Next, I combine the filename provided with each path stored in the "FileLocations" variable using a ForEach loop (I could have extracted the full path including the filename in the previous step, but I find this easier).  
Once I have the full path, I load the content of the file into a variable called "NewContent" using get-content. At the same time I replace the strings in the "NewContent" variable by enclosing the get-content command in brackets and using the "-replace" operator.
The last step is to write the content from the variable back to the file, which I do by piping the "NewContent" variable to the set-content cmdlet
```powershell
# Replace the string in each file found in the specified locations
If ($FileLocations) {
    Try {
        ForEach ($FileLocation in $FileLocations){
            # Combine the base path and file location to create the full path
            $FullPath = $FilePath + "\" + $FileLocation

            # Get content from file and replace string
            $NewContent = (get-content -Path $FullPath -Raw -ErrorAction Stop) -replace $OldString,$NewString 

            # Write new-content to file
            $NewContent | Set-Content -Path $FullPath
        }
    }
    Catch {
        $ErrorMsg = $_.Exception.Message
        Write-Host "Error replacing string: $ErrorMsg"
        Exit 433
    }
}
Else {
    Write-Host "No files have been found"
}
```

The very last part of the script is to stop the transcript if it has been started.
```powershell
# Stop transcript when Transcript parameter is passed.
Try {
    If ($TranscriptPath){
        Stop-Transcript
    }
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Unable to stop transcript: $ErrorMsg"
    Exit 435
}
```

That is all there is to this script.  
As always, if you have any questions or comments, please reach out.