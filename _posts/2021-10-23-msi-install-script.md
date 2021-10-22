---
layout: post
read_time: true
show_date: true
title:  Script to install MSI files in Intune/MEM
date:   2021-10-23 10:00:00 +1000
description: Script for installing MSI applications as Win32 apps in Intune/MEM
img: posts/2021-10-23-msi-install-script/hero.png
tags: [Intune/Endpoint-Manager,PowerShell]
author: Peter Dodemont
published: true
---
This is going to be a brief post about the script I use to install MSI packages as Win32 apps in Intune/MEM. You might be wondering why I would not just install them as line of business apps in Intune/MEM as that takes care of a number of items by itself. The reason is that there are a number of key things you can't do with the line of business apps (e.g. dependencies). I will cover these off in greater detail in a future post.

## The Script
You can find the script in my Github Repo [here](https://github.com/PeterDodemont/Scripts/tree/main/Install-Scripts).
The script follows the same basic premise as all my other scripts. I try and make them as generic as possible and use parameters to pass the values that change. I like this approach as it means I don't need to constantly write and test new scripts.

The first part of the script defines all the parameters you can pass.
```powershell
Param
(
[Parameter(Mandatory=$false)]
[String]
$TranscriptPath
,
[Parameter(Mandatory=$true)]
[ValidateScript({If($_ -NotLike "*.msi"){$true}Else{Throw "The MSI filename should not include the extension."}})]
[String]
$MSIFilename
,
[Parameter(Mandatory=$false)]
[String]
$MSIProperties
)
```
First, there is the "TranscriptPath" parameter, which if passed enables logging of all inputs and outputs to a log file. This is extremely useful for troubleshooting. And passing it as a parameter means I Just need to adjust the install command in Intune/MEM if I want to turn it on or off.
The "MSIFilename" parameter requires just the file name without the extension. It validates this be checking if the supplied file does not end in .msi. The reason for this is that Intune/MEM automatically adds "/qn /norestart" to any install command it detects has a .msi file in it (found this out the hard way :) ).
Finally, the MSIProperties parameter can be used to supply any custom MSI properties as defined/supplied by the developer of the application.

The next section checks if the "TranscriptPath" parameter is passed and if it is, it enables the transcript and saves it to the path passed in the parameter.
```powershell
Try {
    If ($TranscriptPath){
        Start-Transcript -Path "$TranscriptPath\$MSIFilename.log" -Force
    }
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Unable to start transcript: $ErrorMsg"
    Exit 431
}
```

Then I have a variable that adds ".msi" the the msi filename passed, so that it can be used.
```powershell
$MSIInstallFile = $MSIFilename + ".msi"
```

After that the installation is kicked off inserting all the variables generated and provided.
```powershell
Try {
    Start-Process msiexec.exe -ArgumentList "/i $MSIInstallFile /qn /norestart $MSIProperties" -Wait
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "$MSIInstallFile Installation Error: $ErrorMsg"
    Exit 421
}
```

Finally, if the "TranscriptPath" parameter was passed the transcript gets stopped.
```powershell
Try {
    If ($TranscriptPath){
        Stop-Transcript
    }
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Unable to stop transcript: $ErrorMsg"
    Exit 432
}
```

Each section is included in it's on try-catch block and generates a unique error code that can be retrieved from Intune/MEM to assist in troubleshooting even if the transcript is not enabled.

As usual, if you have any questions or comments, please reach out.