---
layout: post
read_time: true
show_date: true
title:  Refreshing the WDAC policy (WDAC management part 2)
date:   2024-08-16 10:00:00 +1000
description: The script that can be used to refresh the WDAC policy on a device.
img: posts/2024-08-17-wdac-refresh-policy/hero.jpg
tags: [powershell]
author: Peter Dodemont
---
This is the 2nd post in my series on managing WDAC through Intune.\
This time around I will be breaking down the script I use on each device that will load and apply my new WDAC policy.\
If you are looking for any of the posts about the other scripts, I'll link them here:
* [WDAC-MergePolicy.ps1](\wdac-policy-merge.html): The script that I use to merge WDAC policy additions into the existing policy.
* WDAC-RefreshPolicy.ps1: This post
* WDAC-CompileAndPackage.ps1: Turning the XML into a binary file that can be deployed and packing it for use by Intune
* WDAC-IntuneUpload: The script that automates the upload, assignment, supersedence, and version management in Intune.

## The Script
You can find the script in my GitHub Repo [here](https://github.com/PeterDodemont/Scripts/tree/main/Install-Scripts).\
As mentioned in the intro, this script will load and apply the latest WDAC policy. It is used as the installation script for my Intune deployments.\
The script will also regenerate .NET native images (since my policies have the Dynamic Code Security Option enabled).

Since this script is used as the installation script for my Intune deployments, it starts with my customary parameters and relevant checks to enable the transcript.\
I only use the transcript when testing or troubleshooting as I don't want logs lingering on the device when not needed. This, along with the ability to easily turn it on and off by just changing the install command in Intune is why I have it set up through a parameter.
```powershell
Param
(
[Parameter(Mandatory=$false)]
[String]
$TranscriptPath
)

# Start transcript when Transcript parameter is passed.
Try {
    If ($TranscriptPath){
        Start-Transcript -Path "$TranscriptPath\RefreshWDACPolicy.log" -Force
    }
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Unable to start transcript: $ErrorMsg"
    Exit 431
}
```

After the transcript section, there is the section with all the variables that can be set or changed.\
First up we have a variable for the filename of the WDAC policy. WDAC requires this filename to be the same as the GUID that is specified in the policy (the GUID is set when the first version of a policy is created).\
The next variable is the name of the executable that will be used for refreshing the policy. This executable needs to be downloaded from Microsoft if you don't already have a copy. RefreshPolicy.exe is the name the executable has when you download it, but you can change it to whatever you like.\
The last variable to set is the location of where the WDAC policies get loaded from. This is a standard location as specified by Microsoft, as such there should be no need to change it.
```powershell
# Set Variables

# Set name of the policy binary file
[String]$PolicyBinaryFilename = "{A754CB49-BA29-433E-8C32-3854DD8590B9}.cip"
# Set filename of the policy refresh tool. This tool will apply all WDAC policies on a device and is provided by Microsoft at this URL https://www.microsoft.com/en-us/download/details.aspx?id=102925
[String]$RefreshPolicyTool = "RefreshPolicy.exe"
# Set destination folder where policy files will be loaded from
[String]$DestinationFolder = $env:WinDir+"\System32\CodeIntegrity\CIPolicies\Active\"
```

Now that the variables are set, I start the actual process of loading and refreshing the policy.\
This process starts by copying the policy file to the location where the WDAC policies are loaded from.\
As usual, I do this operation (and any other operation that could cause an error) in a try-catch block, as I want to create meaningful error messages.
```powershell
# Copy policy file from source to destination
Try {
    Copy-Item -Path $PolicyBinaryFilename -Destination $DestinationFolder -Force
} catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Failed to copy policy to destination folder: $_" -ForegroundColor Red
    Write-host "Detailed error message: $ErrorMsg" -ForegroundColor Red
    Exit 421
}
```

The next step in the process is applying the policy I just loaded. This is accomplished by simply running the RefreshPolicy.exe executable.
```powershell
Try {
    Start-Process -FilePath $RefreshPolicyTool -Wait
} catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Failed to policy refresh tool: $_" -ForegroundColor Red
    Write-host "Detailed error message: $ErrorMsg" -ForegroundColor Red
    Exit 422
}
```

The final step in our process of loading and applying the policy is regenerating the .NET native images for every installed .NET framework version. Native images are part\
For this, I need to run an executable that is installed as part of each .NET framework version called ngen.exe.\
I first look for all the ngen.exe files in the folders .NET framework folders using Get-ChildItem with the -Recurse option and store the output in a variable.\
After that, I can loop through each of the entries in the variable and execute ngen.exe with the update argument to force it to regenerate the native images.\
I'll note that this doesn't seem to solve all of the issues with native images, but it has reduced the errors I get from them significantly.
```powershell
Try {
    # Get ngen utility for installed .NET versions
    $NetNgenVersions = Get-ChildItem  $env:SystemRoot\Microsoft.NET\Framework ngen.exe -Recurse 
    # Run ngen utility for each version of .NET with the update parameter to regenerate images
    ForEach ($NetNgenVersion in $NetNgenVersions) { 
    Start-Process -FilePath $_.FullName -ArgumentList update
    }
} catch {
    $ErrorMsg = $_.Exception.Message
    Write-Host "Failed to regenerate .NET native images: $_" -ForegroundColor Red
    Write-host "Detailed error message: $ErrorMsg" -ForegroundColor Red
    Exit 423
}
```

The final part of the script turns off the transcript if it was set to start using the parameter.
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
    Exit 432
}
```

This script is not very complicated, but it is the heart of the deployment process. Without it, my new policy would not apply to a device or I would end up with a bunch of angry users that have applications that don't work.\
As always if you have any questions or comments, please reach out.