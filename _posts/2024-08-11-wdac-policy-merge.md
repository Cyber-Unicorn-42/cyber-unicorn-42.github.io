---
layout: post
read_time: true
show_date: true
title:  Merging WDAC policy changes (WDAC management part 1)
date:   2024-08-10 10:00:00 +1000
description: Script to add items to the a WDAC XML policy.
img: posts/2024-08-11-wdac-policy-merge/hero.jpg
tags: [powershell]
author: Peter Dodemont
---
Managing WDAC can be quite hard and time-consuming with the tools provided by Microsoft. When I had to start managing WDAC I worked on some scripts that would make the process much easier.
I won't be going through what my process for the deployment is. I have a tool-agnostic blog post on that [here](\app-allow-listing.html)
I also won't cover what is needed to create and deploy the first WDAC policy, Microsoft has documentation on this available [here](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/windows-defender-application-control/design/wdac-design-guide).
What I will be sharing instead is the scripts I used to make the ongoing management and policy updates a lot easier. There will be 4 scripts I will be sharing:
* WDAC-MergePolicy.ps1: This post
* WDAC-RefreshPolicy.ps1: The script used on each device to refresh the policy
* WDAC-CompileAndPackage.ps1: Turning the XML into a binary file that can be deployed and packing it for use by Intune
* WDAC-IntuneUpload: The script that automates the upload, assignment, supersedence, and version management in Intune.

## The Script
You can find the script in my GitHub Repo [here](https://github.com/PeterDodemont/Scripts/tree/main/Misc).
This script will add the XML files provided to existing XML policy files. Unless a specific policy version is specified it will grab the latest policy version XML file and add the the config from the specified XML files. The script will create both audit mode (AM) and enforced mode (EM) XML policy files. It will only run against 1 specific group at a time. This is because I need to be careful and precise about what files get added to the policy. I want to reduce the risk of accidentally adding files to a policy that shouldn't have it.

As with most of my other scripts, I have parameters set up to take all the variables that change with each run or provide options for script execution.
There are 2 mandatory parameters, 1 for the IDs used for the groups in your deployment (CohortID) and 1 for the XML files that will be added to the config (AddToScan).
then there are also 2 optional parameters, 1 to set a specific old policy version number to use (instead of the latest version found), and 1 to specify the new policy version number to use (this is useful if you want to jump major versions).
```powershell
param (
 [Parameter(Mandatory=$false)]
 [decimal] $OldBasePolicyVersion,
 [Parameter(Mandatory=$false)]
 [decimal] $NewBasePolicyVersion,
 [Parameter(Mandatory=$true,Position=0)]
 [ValidateSet("1", "2", "3")]
 [string] $CohortID,
 [Parameter(Mandatory=$true,Position=1)]
 [array] $AddToScan
)
```

After the parameters there are a number of variables that are being set, these will need to be customized to fit your needs. They are set as variables instead of parameters, as these should stay the same for each run. You will need to update these variables so they suit your environment
Before digging into the variables and what they do, I want to note that I have all WDAC-related files in a folder structure with sub-folders. There are folders for the scripts, policy XML files, policy binary files, additional XML files to be added, and so on.
The first 2 variables, PolicyPathAM and PolicyPathEM, are the relative paths to the folders containing the policy XML files. I have a separate folder for each policy type for each different cohort.
The next variable, ApplicationXMLPath, sets the relative path to the folder containing the XML files to be added. In my setup, this folder actually contains sub-folders for each application to make management easier, but that is not required.
Then, I get the date in the year-month-day format so it can be used in file names.
The last set of variables, NewPolicyBaseNameAM and NewPolicyBaseNameEM, specify the fixed part of each policy XML filename.
```powershell
# Set variables
# Set paths for policy XML files
[String]$PolicyPathAM = "..\Policies_AM_Cohort"+$CohortID+"\"
[String]$PolicyPathEM = "..\Policies_EM_Cohort"+$CohortID+"\"

# Set path to folder containing XML files to be added
[String]$ApplicationXMLPath = "..\Application scans\"

# Get the current date for use in file names
$CurrentDate = Get-Date -Format "yyyy-MM-dd"

# Set the the policy base filenames
[String]$NewPolicyBaseNameAM = "CyberUnicorn_WDAC_AM_Cohort"
[String]$NewPolicyBaseNameEM = "CyberUnicorn_WDAC_EM_Cohort"
```

Next, I create some arrays that will contain the names of the XML files that were merged and the ones that weren't. These are specified here as they need to persist through the entire script, if I were to create them when I first need them, they would be cleared once the script exits the for-each loop.
```powershell
# Create Array to contain missing and merged XML files
$MissingXMLFiles = @()
$MergedXMLFiles = @()
```

Now that all variables are set, I first check to see if an old policy version was specified through its parameter, OldBasePolicyVersion. If an old policy version number was specified, I parse the number into a a version number.
If the old policy version is not specified I get the version number from the most recent audit mode policy file (I keep audit and enforced mode policies identical as that makes management a lot easier).
First, I get the filename without the extension of the most recent file in the audit mode policy path specified earlier.
Then, I split the file so only the version number remains.
Next, I remove "v" so I am left with a number one.
Finally, I parse the number into a version number.
```powershell
# Check if old base policy version was provided, if not get the latest policy version from the most recently modified file
If ($OldBasePolicyVersion) {
    # Parse the version numbers into decimal format to be manipulated later
 [Decimal]$OldBasePolicyVersion = [Decimal]::Parse("{0:0.00}" -f ($OldBasePolicyVersion))
} Else {
    # If no policy version to supersede was provided, get the filename without extension of the most recently modified file for this cohort, using audit mode policy as base.
 [String]$OldPolicyVersionBasePath = (Get-ChildItem -Path $PolicyPathAM -File -Recurse -ErrorAction SilentlyContinue | sort -Property LastWriteTime -Descending | Select -First 1).BaseName
    # Split the filename so only the version remains
 [String]$OldBasePolicyVersion = $OldPolicyVersionBasePath -split ("_") | Select -Last 1
    # Remove the "v" character from remaining information to get just the version number
 [String]$OldBasePolicyVersion = $OldBasePolicyVersion -Replace "v"
    # Parse the version numbers into decimal format to be manipulated later
 [Decimal]$OldBasePolicyVersion = [Decimal]::Parse("{0:0.00}" -f ($OldBasePolicyVersion))
}
```

In the next step, I check if a new policy version number was provided in the parameter. If it has, I parse the number into a version number. If it hasn't I add 0.01 to the old version number from the previous step.
```powershell
# Check if new policy version was provided, if not increment old policy version by 0.01 for new version number
If ($NewBasePolicyVersion) {
    # Parse the version numbers into decimal format to be manipulated later
 [Decimal]$NewBasePolicyVersion = [Decimal]::Parse("{0:0.00}" -f ($NewBasePolicyVersion))
} else {
    # Set new policy version to old policy version incremented by 0.01 and parse the version numbers into decimal format to be manipulated later
 [Decimal]$NewBasePolicyVersion = [Decimal]::Parse("{0:0.00}" -f ($OldBasePolicyVersion + 0.01))
}
```

Now we get to the main part of the script. This starts off with a for-each loop, so we can perform the actions on each XML file specified in the parameter.
```powershell
# Iterate through each XML to be added to see they exist 
foreach ($SoftwareScanToAdd in $AddToScan) {
```

The first step in the loop is setting the full path to the XML file to add to the policy.
```powershell
# Set variable containing full path to the XML file
$ApplicationToAddFullName = $ApplicationXMLPath+$SoftwareScanToAdd
```

Then, I check if the file exists or not. If it doesn't exist write a message to the screen and add the path as specified in the parameter to the array of files that were missing. 
```powershell
    # Test if XML paths exist
    if (-not (Test-Path -Path $ApplicationToAddFullName)) {
        Write-Host "XML file '$ApplicationToAddFullName' does not exist, skipping merge." -ForegroundColor Red

        # Add file to array of missing files
        $MissingXMLFiles += $SoftwareScanToAdd
```

If the file does exist, confirm this with a message on the screen and start the merge process.
The merge process starts by setting the search string used for the path to the old policy file to use. This uses the audit mode path and old policy versions specified earlier.
Next, I do a search for that file and grab the full path to that file. Once I have the filename I write a message to the screen with the full path to the old policy file.
Then, I set the names that will be used for the new policy files and inside the XML by combining, the base name, the cohort ID, the date, and the version number.
The final step in the preparation is setting the full paths for both audit and enforced mode files by combining the variable with the paths, names from the previous step, and the file extension (.xml).
```powershell
 }else{
        Write-Host "XML file '$ApplicationToAddFullName' exists and will be merged." -ForegroundColor Green
        
        # Set the search string old policy path (Using Audit Mode policy as base)
        $OldPolicyPathSearchString = $PolicyPathAM+"*"+$OldBasePolicyVersion+"*"
        # Get old policy file name (Using Audit Mode policy as base)
        $OldPolicyName = Get-ChildItem –Path $OldPolicyPathSearchString -File -Force -ErrorAction SilentlyContinue
        $OldBasePolicyFileXML = $OldPolicyName.FullName
        # Display old policy filename (Using Audit Mode policy as base)
        Write-Host "Old base policy XML filename (Audit mode policy used as base):" -ForegroundColor Magenta
        Write-Host $OldBasePolicyFileXML -ForegroundColor Magenta
        # Set names of new policies
        $NewPolicyNameAM = $NewPolicyBaseNameAM+$CohortID+"_"+$CurrentDate+"_v"+$NewBasePolicyVersion
        $NewPolicyNameEM = $NewPolicyBaseNameEM+$CohortID+"_"+$CurrentDate+"_v"+$NewBasePolicyVersion
        # Set full path to new policy files
        $NewBasePolicyAMFileXML = $PolicyPathAM+$NewPolicyNameAM+".xml"
        $NewBasePolicyEMFileXML = $PolicyPathEM+$NewPolicyNameEM+".xml"
```

With all the preparation work done, I can start the process of merging the XML file into the existing policy.
I output a message the merge is starting and that this process can take some time (the larger the policy the longer this step takes), then I start the merge.
I suppress the output of this command by sending it to "Out-Null"
Next, I modify the version information and name inside the XML file commands. This information is reported back into the logs, so updating it makes ongoing management easier.
```powershell
        Try {
            # Merge new XML file into old policy for audit mode
            Write-Host "Starting policy merge, this could take a while to complete" -ForegroundColor DarkCyan
            Merge-CIPolicy -PolicyPaths $OldBasePolicyFileXML, $ApplicationToAddFullName -OutputFilePath $NewBasePolicyAMFileXML | Out-Null
            # Set policy version and name on the new Audit Mode policy file (can be used when looking at the logs)
            Set-CIPolicyVersion -FilePath $NewBasePolicyAMFileXML -Version $NewBasePolicyVersion
            Set-CIPolicyIdInfo -FilePath $NewBasePolicyAMFileXML -PolicyName $NewPolicyNameAM
 } Catch {
            $ErrorMsg = $_.Exception.Message
            Write-Host "Error creating Audit mode policy for xml $ApplicationToAddFullName" -ForegroundColor Red
            Write-host "Detailed error message: $ErrorMsg" -ForegroundColor Red
            Return
 }
```

With the audit mode policy merged, I copy the audit mode policy to the enforced mode policy folder while simultaneously renaming it.
I then change the name inside the XML to indicate that this is the enforce mode policy.
Finally, I change the option in the XML file that indicates if the policy is for audit more or enforced mode.
```powershell        
        Try {
            # Copy and rename Audit Mode policy to Enforce Mode folder
            Copy-Item -Path $NewBasePolicyAMFileXML -Destination $NewBasePolicyEMFileXML -Force
            # Set policy name on the new Enforce Mode policy file (can be used when looking at the logs)
            Set-CIPolicyIdInfo -FilePath $NewBasePolicyEMFileXML -PolicyName $NewPolicyNameEM
            # Change the policy file from Audit Mode to Enforce Mode
            Set-RuleOption -FilePath $NewBasePolicyEMFileXML -Option 3 -Delete
 } Catch {
            $ErrorMsg = $_.Exception.Message
            Write-Host "Error creating Enforce mode policy for xml $ApplicationToAddFullName" -ForegroundColor Red
            Write-host "Detailed error message: $ErrorMsg" -ForegroundColor Red
            Return
 }
```

The are a few steps left before we can start the loop again.
First, I add the path to the XML file, as specified in the parameter, to the array of files that were merged.
Then we pause for 10 seconds to let the device settle.
Finally, we set the old version to be the version we just created and we set the new version number by adding 0.01 to the version we just created
```powershell
        # Add file to array containing files that were merged
        $MergedXMLFiles += $SoftwareScanToAdd

        # Sleep for 10 seconds
        Start-Sleep -Seconds 10

        # Increment policy version numbers
        $OldBasePolicyVersion = [Decimal]$NewBasePolicyVersion
        $NewBasePolicyVersion = [Decimal]::Parse("{0:0.00}" -f ($NewBasePolicyVersion + 0.01))
 }
}
```

After the loop completes and no more files are left to be merged, there are a few final steps.
Firstly, I set the final policy version to be the old policy version as set from the last loop.
Next, I check if there are any files in the array with the XML files that were not found. If there is, I display a message with the details of the files that were missing.
Finally, I check if there are any files in the array with the XML files that were merged. If there is, I display a message confirming the files that were merged and what the final policy version ended up being.
```powershell
# Set final policy version number
$FinalPolicyVersion = $OldBasePolicyVersion

# Display message with failed XML files
if ($MissingXMLFiles) {
    Write-Host "The following XML files were NOT merged: $MissingXMLFiles" -ForegroundColor Red
    Write-Host "Please check these files exist and re-run merge for these files." -ForegroundColor Red
}
# Display message with successful XML files
if ($MergedXMLFiles) {
    Write-Host "The following XML files were merged: $MergedXMLFiles" -ForegroundColor Green
    Write-Host "The final policy version is $FinalPolicyVersion" -ForegroundColor Cyan
}
```

That brings us to the end of the script. I hope this sheds some light on how I manage the process of updating the WDAC policy.
As always if you have any questions or comments, please reach out.