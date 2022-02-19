---
layout: post
read_time: true
show_date: true
title:  Replacing a string in a file with PowerShell
date:   2022-02-12 10:00:00 +1000
description: Script to replace a string in a configuration file and restart a service.
img: posts/2022-02-12-replace-and-restart/hero.png
tags: [powershell]
author: Peter Dodemont
---
A little while ago I had to make some changes to some configuration files. The change in question was updating an ip address in some configuration files and replace it with a dns name. I had to make these changes as I had to update the IP of a Splunk server during a re-ip project. This had to be done in several configuration files on all servers, so I decide the script it rather than manually making the change on each server.
(If you know how Splunk works you might think that I could have just made the change on the deployment server and have that pushed out, and you would be right, unfortunately as part of the re-ip process the routing to the deployment server was changed and reverting those changes would be more work and have a larger impact.)

## The Script
You can find the script in my GitHub Repo [here](https://github.com/PeterDodemont/Scripts/tree/main/Misc).
As with most of my other scripts, I have parameters setup to take all the variables required. This allows me to easily re-use this script in the future.
There are parameters for the old and new strings as well as an array for the locations of the files. There is also a parameter for a process name and a service name who's use I'll cover further down. And finally, there is my customary transcript path parameter that can be used for troubleshooting if you need to see the exact output you'd get if running it through PowerShell directly.
```powershell
Param
(
[Parameter(Mandatory=$false)]
[String]
$TranscriptPath
,
[Parameter(Mandatory=$true)]
[string[]]
$FileLocations=@()
,
[Parameter(Mandatory=$true)]
[string]
$OldString
,
[Parameter(Mandatory=$true)]
[string]
$NewString
,
[Parameter(Mandatory=$false)]
[string]
$ProcessName
,
[Parameter(Mandatory=$false)]
[string]
$ServiceName
)
```

After the parameters I check if the transcript path has been passed and if it has, I start the transcript and save it to the specified path.
```powershell
# Start transcript when Transcript parameter is passed.
Try {
    If ($TranscriptPath){
        Start-Transcript -Path "$TranscriptPath\ReplaceAndRestart.log" -Force
    }
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Unable to start transcript: $ErrorMsg"
    Exit 431
}
```

Then, I start the process to replace the string in each of the files provided. First, I read the content of the file using get-content into a variable. Using "-Raw" ensures I have the raw text content of the file and not any formatted versions. Using the "-replace" parameter I replace the old string with the new string every time it is encountered after the content is read. I could have done this in 2 steps, but by enclosing the get-content command in brackets I ensure this is run first before the -replace is run on the result of the get-content command.
I then save the content from the variable back over the existing file.
Everything is in a Try-Catch block for error handling purposes.
```powershell
# Replace the string in each of the specified locations
Try {
    ForEach ($FileLocation in $FileLocations){
        # Get content from file and replace string
        $NewContent = (get-content -Path $FileLocation -Raw -ErrorAction Stop) -replace $OldString,$NewString 

        # Write new-content to file
        $NewContent | Set-Content -Path $FileLocation
    }
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Error replacing string: $ErrorMsg"
    Exit 432
}
```

Next, I check if a process name was passed and if it has, I force stop the process with that name. As before, the commands are enclosed in a try-catch block.
```powershell
# Force stop process
Try{
    If ($ProcessName) {Stop-Process -Name $ProcessName -Force -ErrorAction Stop}

    # Sleep for 10 seconds to allow the process to be stopped
    Start-sleep -s 10
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Error stopping process: $ErrorMsg"
    Exit 433
}
```

The next step is to restart the service if one was passed to the parameter.
```powershell
# Restart service
Try {
    If ($ServiceName){Restart-Service -Name $ServiceName -Force -ErrorAction Stop}
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Error restarting service: $ErrorMsg"
    Exit 434
}
```

And finally, I will check if a path for the transcript was passed, and if it was, I stop the transcript.
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

And that is all there is too it. As always if you have any questions or comments please reach out.