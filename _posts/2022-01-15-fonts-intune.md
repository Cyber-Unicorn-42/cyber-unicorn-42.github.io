---
layout: post
read_time: true
show_date: true
title:  Installing fonts in Intune/MEM
date:   2022-01-15 10:00:00 +1000
description: Script for installing fonts in Intune/MEM
img: posts/2022-01-15-fonts-intune/hero.png
tags: [intune/endpoint-manager,powershell]
author: Peter Dodemont
---
Every once in a while I get asked to install some new fonts on all devices. Doing it locally on devices is pretty easy, copy them to the fonts folder and you're done. Doing it via Intune/MEM, you need to manually add a registry entry for each font you want to install.

## The Script
As usual the script can be found in my Github Repo [here](https://github.com/PeterDodemont/Scripts/tree/main/Install-Scripts).
As in most of my scripts the fonts to install get passed to the script via a parameter. The parameter is an array that will accept multiple filenames.
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

The first step is to copy the font to the fonts folder in the windows install directory. Using the Windir environment variable ensures that the script works, even if the windows directory is not in the default location of C:\Windows.
```powershell
# Copy font to fonts directory
Copy-Item -Path $Font -Destination $Env:Windir\Fonts -Force
```

Next, I split the filename into the name and the extension and save each in a variable for use later in the script.
```powershell
# Get font name and type
$FontSplit = $Font.split(".")
$FontName = $FontSplit[0]
$FontType = $FontSplit[1]
```

After splitting the filename, I use the variable with the extension to determine if the font is a TrueType or an OpenType font, and save a specific string in a variable to use when creating the registry entry. I use a switch statement as it can only ever be one of the two. This way if it matches on the first case it will not process the rest of the switch statement.
```powershell
# Set value for the font type in the registry
Switch ($FontType) {
    "ttf" {
        $FontTypeName = "(TrueType)"
        Break
    }
    "otf" {
        $FontTypeName = "(OpenType)"
        Break
    }
}
```

The final step is to create the registry entry using the variables created earlier in the script.
```powershell
# Create registry entry for the font
New-ItemProperty -Value "$Font" -Path "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Fonts" -PropertyType string -Name "$FontName $FontTypeName" -Force > $null
```

Because the parameter can accept multiple fonts I need to run the steps through a foreach loop, so that each step happens for each font specified.
```powershell
ForEach ($Font in $Fonts) {
# Copy font to fonts directory
...
# Get font name and type
...
# Set value for the font type in the registry
...
# Create registry entry for the font
...
}
```

These are all the steps you will need to create your own script to install fonts in Intune/MEM. If you look at the script in my GitHub repo, you'll notice that as with most of my other scripts it includes a transcript option as well.
There are still improvements that can be made to the script (e.g. validating the variable only contains filenames with valid extensions). Maybe one day I'll find I'll need them, but for now I will leave it as is, since the script works very reliably.

As always if you have any questions or comments, feel free to reach out.