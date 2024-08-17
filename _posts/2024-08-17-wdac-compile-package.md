---
layout: post
read_time: true
show_date: true
title:  Compiling and packaging WDAC policies (WDAC management part 3)
date:   2024-08-17 10:00:00 +1000
description: The script that I use for compile WDAC policies and packaging them so they can be used by Intune.
img: posts/2024-08-17-wdac-compile-package/hero.jpg
tags: [powershell]
author: Peter Dodemont
---
This is the 3rd post in my series on managing WDAC through Intune.\
This post focuses on the script I use to compile my the WDAC policies for all enforcement modes and cohorts./
The other post in this series can be found below:
* [WDAC-MergePolicy.ps1](\wdac-policy-merge.html): The script that I use to merge WDAC policy additions into the existing policy.
* [WDAC-RefreshPolicy.ps1](\wdac-refresh-policy.html): The script used on each device to refresh the policy
* WDAC-CompileAndPackage.ps1: This post
* WDAC-IntuneUpload: The script that automates the upload, assignment, supersedence, and version management in Intune.

## The Script
You can find the script in my GitHub repo [here](https://github.com/PeterDodemont/Scripts/tree/main/Misc).\
This script is the one I use after I have merged my additions into a new policy. It finds the latest version of the policy for each enforcement level and cohort ID and then compiles it into the binary format used by WDAC.\
After it has compiled the policy, it packages the policy, and the required scripts and files into an intunewin package that can be uploaded to Intune.\

The script starts with defining a number of parameters.\
There are 2 mandatory parameters, CohortIDs and EnforcementLevels, and 1 optional one, PolicyVersionToCompile.\
CohortIDs, is used to specify which cohorts you want to run the script for, and EnforcementLevels, is used to specify which enforcement levels the script will run for.\
The script will run against each combination of specified CohortIDs and EnforcementLevels. E.g. if you specify 1 and 3 for the cohort IDs and AM and EM for the enforcement levels, the script will run for cohort 1 AM, cohort 1 EM, cohort 3 AM, and cohort 3 EM.\
PolicyVersionToCompile, is used if I need to compile a specific version of a policy (useful if I merged multiple files and the policy doesn't load, I can step back through each version to see when it breaks).\
```powershell
# Set parameters
param (
    [Parameter(Mandatory=$false)]
    [string] $PolicyVersionToCompile,
    [Parameter(Mandatory=$true,Position=1)]
    [ValidateSet("AM", "EM")]
    [array] $EnforcementLevels,
    [Parameter(Mandatory=$true,Position=0)]
    [ValidateSet("1","2","3")]
    [array] $CohortIDs
    )
```

Next, I set a number of variables, some of these will look familiar if you have read the other posts in the series.\
The first variable I set, IntunePackagingFolder, points to the relative path where all the files will be located that will be packaged into the intunewin package file. This includes the refresh policy script that was the subject of the [previous article](\wdac-refresh-policy.html).\
Then, I set the PolicyID variable. This variable contains the GUID that is specified in the policy file (the GUID is set when the first version of a policy is created).\
After that, I set the name of the Intune packaging application. This file is located in the same folder this script is executed from.\
The variable, ArgumentList, contains the arguments that will be used when running the Intune packaging application. It uses some of the variables specified earlier.\
I set the PackagedPathAndFilename variable next. This is the path and filename of the package as created by the Intune packaging application.\
The following variable, BasePolicyPathStart, sets the start of the relative path to where the policy XML and binary files a stored (I have the folders start with the same name as I find that the easiest).\
The next 2 variables, BasePolicyXMLPathEnd and BasePolicyBinaryPathEnd, set the end of the path where the XML and binary policy files are stored respectively. I store the policies per enforcement level, so further down the script the enforcement level is inserted in between these variable and the BasePolicyPathStart one.\
The IntunePackageDestinationFolderStart variable, is used to set the start of the relative path to where I keep a copy of all the WDAC Intune packages I have created.\
The last variable I set, RenamedPackagedFilenameStart, is the start of the filename of the files located in the folder set in the previous variable.
```powershell
# Set variables
# Path to intune packaging folder
[string]$IntunePackagingFolder = "..\Intune_Deploy_Tools\"
# GUID of the base policy (used in script installing policy on each device). Update with a unique GUID for your deployment
$PolicyID = "{A754CB49-BA29-433E-8C32-3854DD8590B9}"
# Name of Intune packaging executable
$IntunePackagingTool=".\IntuneWinAppUtil.exe"
# Argument list used by Intune packaging tool
$ArgumentList = "-c $IntunePackagingFolder -s $IntunePackagingFolder"+"WDAC-RefreshPolicy.ps1 -o $IntunePackagingFolder -q"
# Packaged path and filename
$PackagedPathAndFilename  = $IntunePackagingFolder+"WDAC-RefreshPolicy.intunewin"
# Set the beginning of the path to the folder containing policy XML and Binary files
$BasePolicyPathStart = "..\Policies_"
# Set the end of the path to the folder containing policy XML and Binary files
$BasePolicyXMLPathEnd = "_Cohort"
$BasePolicyBinaryPathEnd = "_BIN_Cohort"
# Set the beginning of the path to the folder containing Intune packaged files
$IntunePackageDestinationFolderStart = "..\Intune_Package_"
# Set the beginning of the filename for the Intune package file that will be stored in the
$RenamedPackagedFilenameStart = "DeployWDACPolicyCohort"
```

Before I kick the script off, I check whether a specific policy version has been given. If a specific version was passed along with multiple cohortIDs, the script errors out.\
The reason being that it that it is highly unlikely that multiple cohorts are on the same version and I don't want to accidentally revert the wrong cohort to a specific previous version.
```powershell
# Check if multiple cohorts and a policy version to compile has also been specified. If they are, provide error.
If ((($CohortIDs.Count) -gt 1)-and ($PolicyVersionToCompile)) {
    # Output message
    Write-Host "Specifying multiple cohorts does not support specifying a policy version to compile." -ForegroundColor Red
    Return
}
```

With all the global variables set and the safety check done, I start with a for-each loop that loops through each enforcement mode specified.\
The first step in the enforcement level loop, is setting variables that will remain static throughout the loop, but change between enforcement levels.\
There are 3 variables I set, BasePolicyXMLPath, BasePolicyBinaryPath, and IntunePackageDestinationFolder.\
All 3 are paths and they add the enforcement levels into the path at the correct point as per my folder structure.\
BasePolicyXMLPath, is the start of the path to where the XML policy files are located.\
BasePolicyBinaryPath, is the start of the path to where the binary policy files are located.\
IntunePackageDestinationFolder, is the full relative path to where I keep a copy of all the WDAC Intune packages.
```powershell
Foreach ($EnforcementLevel in $EnforcementLevels) {
    # Set Variable
    # Base path to policy XML file for each enforcement level
    [string]$BasePolicyXMLPath = $BasePolicyPathStart + $EnforcementLevel + $BasePolicyXMLPathEnd
    # Base path to policy binary file for each enforcement level
    [string]$BasePolicyBinaryPath = $BasePolicyPathStart + $EnforcementLevel + $BasePolicyBinaryPathEnd
    # Destination folder of the Intune packaged files
    [string]$IntunePackageDestinationFolder = $IntunePackageDestinationFolderStart + $EnforcementLevel + "\"
```

After the enforcement mode variables are set, I start the cohort ID for-each loop.\
I start by setting some path variables, FullPolicyXMLPath and FullPolicyBinaryPath. They combine variables set for each enforcement level with the cohort ID to construct the final folder paths for the policy XML and Binary files.
```powershell
# Run process for each provided cohort
    ForEach ($CohortID in $CohortIDs) {
        # Set variables
        # Full path to policy XML file for each cohort
        [string]$FullPolicyXMLPath = $BasePolicyXMLPath + $CohortID + "\"
        # Full path to policy binary file for each cohort
        [string]$FullPolicyBinaryPath = $BasePolicyBinaryPath + $CohortID + "\"
```

The next section of the cohort ID loop sets a variable, PolicyXMLFile, with details of the XML policy file that will be compiled into binary format.\
First I validate if a version was specified. If it was, I search the XML policy folder for that version and set the PolicyXMLFile variable with the details of that file.\
If no version is specified, I search the XML policy folder for the file that was modified last, and set the PolicyXMLFile variable to the details of that file.\
In both cases, I also set the NewPolicyVersionToCompile variable as that will be used later in the script whenever I need to refer to the new version.\
If a version was specified it's a straightforward copy of the PolicyVersionToCompile variable.\
If no version was specified I get the filename without the extension and keep only the characters after the last underscore. Next, I remove the letter "v", so I'm left with only the number.\
I can do it this way because I have standardized the filenames I use.
```powershell
        # Check if policy version was provided
        if ($PolicyVersionToCompile) {
            # Get specific policy version XML file from folder with XML files
            $PolicyXMLFile  = Get-ChildItem -Path $FullPolicyXMLPath -Include *$PolicyVersionToCompile* -File -Recurse -ErrorAction SilentlyContinue
            [string]$NewPolicyVersionToCompile = $PolicyVersionToCompile
        } else {
            # Get latest policy XML file from folder with XML files
            $PolicyXMLFile  = Get-ChildItem -Path $FullPolicyXMLPath -File -Recurse -ErrorAction SilentlyContinue | Sort LastWriteTime | select -last 1
            # Get policy version from the filename by first spit by the last underscore character followed by removing the letter v
            [string]$NewPolicyVersionToCompile = ($PolicyXMLFile.BaseName).split("_")[-1]
            [string]$NewPolicyVersionToCompile = $NewPolicyVersionToCompile -Replace "v"
        }
```

Now that I have a variable with file details, I check to see if that variable is empty. If it is, I stop the script and throw an error to advise there was an issue getting the XML policy file details.
```powershell
        If (!($PolicyXMLFile)){
            Write-host "Error getting details of policy XML file" -ForegroundColor Red
            Return
        }
```

Next, I set more variables.\
The first 2 variables, FullPathToPolicyXML and BinaryPolicyPathAndFilename, use details from the file gathered just before to set the full path to the policy XML file and the path and filename to the binary policy file, respectively.\
The next variable, IntuneBinaryPolicyPathAndFilename, sets the path and filename for the binary policy file that will be included in the Intune package.\
The final variable, RenamedPackagedPathAndFilename, is the full path and filename to the Intune package file. I add the enforcement level, cohort ID, and version number to the filename, so it can be easily identified in the future.
```powershell
        # Set variables that include details gathered above
        # Full path, including filename, to policy XML file 
        $FullPathToPolicyXML = $PolicyXMLFile.FullName
        # Path and filename for binary policy file
        $BinaryPolicyPathAndFilename = $FullPolicyBinaryPath + $PolicyXMLFile.BaseName + ".cip"
        # Path and filename with GUID for Intune package. As per Microsoft documentation: Policy binary files should be named as {GUID}.cip for multiple policy format files (where {GUID} = <PolicyId> from the Policy XML)
        $IntuneBinaryPolicyPathAndFilename  = $IntunePackagingFolder + $PolicyID + ".cip"
        # Path and filename of the renamed Intune package
        $RenamedPackagedPathAndFilename = $IntunePackageDestinationFolder + $RenamedPackagedFilenameStart + $CohortID + $EnforcementLevel + $NewPolicyVersionToCompile + ".intunewin"
```

With all the variables set, I start by converting the XML policy into the binary format supported by WDAC.\
This is done using the built-in ConvertFrom-CIPolicy command and passing along the full path to the XML file and the full path where the binary file will be stored.\
The binary file is saved directly to the folder where I keep a copy of all the files I generated.\
As usual, this and all other commands that could cause the script to error out, are enclosed in a try-catch block, so that I can give more meaningful error messages.
```powershell
        # Generate new WDAC Policy binary file for cohort and Enforcement level.
        Try {
            ConvertFrom-CIPolicy -XmlFilePath $FullPathToPolicyXML -BinaryFilePath $BinaryPolicyPathAndFilename
            Write-Host "Binary policy file for $EnforcementLevel cohort $CohortID version $NewPolicyVersionToCompile successfully generated" -ForegroundColor Magenta
        } Catch {
            $ErrorMsg = $_.Exception.Message
            Write-Host "Error generating WDAC policy" -ForegroundColor Red
            Write-host "Detailed error message: $ErrorMsg" -ForegroundColor Red
            Return
        }
```

After the binary file has been generated, the newly generated file is copied to the folder that contains the files to be packaged.\
The file is also renamed during the copy to the GUID as is required by Microsoft.
```powershell
        # Copy generated binary policy to Intune packaging folder
        Try {
            
            Copy-Item -Path $BinaryPolicyPathAndFilename -Destination $IntuneBinaryPolicyPathAndFilename -Force
            Write-Host "Binary policy file for $EnforcementLevel cohort $CohortID version $NewPolicyVersionToCompile successfully copied to Intune packaging folder" -ForegroundColor Magenta
        } Catch {
            $ErrorMsg = $_.Exception.Message
            Write-host "Error copying binary policy file to Intune packaging folder" -ForegroundColor Red
            Write-host "Detailed error message: $ErrorMsg" -ForegroundColor Red
            Return
        }
```

With the binary file copied, the packing folder now contains all the files it needs.\
The files in that folder are:
* [The refresh script](\wdac-refresh-policy.html)
* [The refresh tool provided by Microsoft](https://www.microsoft.com/en-us/download/details.aspx?id=102925)
* The binary policy file

The next step is running the Intune packaging tool with the arguments specified at the start of the script.\
This will generate the intunewin package file I can upload to Intune later.
```powershell
        # Run Intune packaging tool
        Try {
            (Start-Process -FilePath $IntunePackagingTool -ArgumentList $ArgumentList -PassThru:$true -ErrorAction Stop -NoNewWindow).WaitForExit()
            Write-host "Intune package for $EnforcementLevel cohort $CohortID version $NewPolicyVersionToCompile successfully generated" -ForegroundColor Magenta
        } Catch {
            $ErrorMsg = $_.Exception.Message
            Write-Host "Error running Intune packaging tool" -ForegroundColor Red
            Write-host "Detailed error message: $ErrorMsg" -ForegroundColor Red
            Return
        }
```

After the Intune packaging tool has run, I move the intunewin file to the folder that holds a copy of each version I have packaged.\
During the move the file is also renamed, making identifying what cohort, enforcement level, and version the file is for easier.
```powershell
        # Move the packaged file to final destination
        Try {
            Move-Item -Path $PackagedPathAndFilename -Destination $RenamedPackagedPathAndFilename -Force
            Write-Host "Intune package for $EnforcementLevel cohort $CohortID version $NewPolicyVersionToCompile successfully moved to final destination" -ForegroundColor Magenta
        } Catch {
            $ErrorMsg = $_.Exception.Message
            Write-Host "Error moving Intune package to final destination" -ForegroundColor Red
            Write-host "Detailed error message: $ErrorMsg" -ForegroundColor Red
            Return
        }

```

The final step of the cohort ID loop is removing variables that could cause issues in the next loop.\
```powershell
        # Remove variables that could cause issues on next run
        Remove-Variable FullPolicyXMLPath -Force
        Remove-Variable FullPolicyBinaryPath -Force
        Remove-Variable PolicyXMLFile -Force
        Remove-Variable FullPathToPolicyXML -Force
        Remove-Variable BinaryPolicyPathAndFilename -Force
        Remove-Variable IntuneBinaryPolicyPathAndFilename -Force
        Remove-Variable RenamedPackagedPathAndFilename -Force
        Remove-Variable NewPolicyVersionToCompile -Force

        # Display final success message
        Write-Host "$EnforcementLevel Cohort $CohortID version $NewPolicyVersionToCompile successfully generated and packaged" -ForegroundColor Green
```

After the cohort ID loops have completed for a specific enforcement level, I remove the variables that could cause issues in the next loop of the enforcement level loops.\
```powershell
    # Remove variables that could cause issues on next run
    Remove-Variable BasePolicyXMLPath -Force
    Remove-Variable BasePolicyBinaryPath -Force
    Remove-Variable IntunePackageDestinationFolder -Force
```

Once all enforcement level loops have completed, I print out a final message advising everything completed successfully.\
You will have noticed I also output messages at different points throughout the script, which I didn't cover in the explanations of each step.\
This is because these are status messages that don't offer any additional functionality other than what step just completed.
```powershell
# Display final completion message
Write-Host "All policies successfully generated and packaged" -ForegroundColor DarkGreen
```

This concludes the post on the compile and package script. And while the script seems complicated this is mostly because there are a lot of variables that are being set and constructed.\
Once all of those are done, the commands being run aren't very complicated.\
As always if you have any questions or comments, please reach out.