---
layout: post
read_time: true
show_date: true
title:  Configuring Windows Hello through Intune/MEM
date:   2021-11-06 10:00:00 +1000
description: How can you setup up Windows Hello through Intune/MEM (not Windows Hello For Business).
img: posts/2021-11-06-windows-hello-proactive/hero.jpg
tags: [intune/endpoint-manager,powershell]
author: Peter Dodemont
---
A little while ago I found myself in a position where I wanted to enable biometric login as that is more convenient to use that username and password. As I work in an org where we use Microsoft Surface Pro devices, Windows Hello came to mind straight away. Unfortunately we were unable to use Windows Hello for Business at the time as we did not have a Server 2019 DC (something that has now been rectified :) ). So, I looked into trying to configure Windows Hello (the consumer version instead).
I quickly came to the discovery that you can only configure Windows Hello for Business settings through MEM. You used to be able to configure the Windows Hello settings, but those were removed. I came across several articles that went down the path of configuring a custom template to set the required settings, but I was not successful using that path. So, I started looking at other options.
I know that you can configure the settings through Group Policy. Which indicates that there are sections in the registry that can be modified to achieve the desired result. A quick search provided me with all the keys that needed to be set, now I all needed was a delivery method. I immediately thought of my new favourite feature of MEM, the proactive remediations. Below is the 2 scripts I wrote to be able to configure all the Windows Hello settings and validate they are applied correctly.
The scripts can be found on [My Github Repo](https://github.com/PeterDodemont/Scripts). The detection script is [here](https://github.com/PeterDodemont/Scripts/blob/main/Intune/ProactiveRem-RegHelloLocal-Detection.ps1) and the configuration script is [here](https://github.com/PeterDodemont/Scripts/blob/main/Misc/Set-RegHelloLocal.ps1).

## The Configuration Script
The configuration script begins by setting some variable used for creating the registry keys with the correct name in the correct path. I always prefer to use variables where possible as it allows me to repurpose script if I need to in the future.
```powershell
$RegKeyPath = "HKLM:\SOFTWARE\Policies\Microsoft\PassportForWork\PINComplexity"
$DigitsRegKey = "Digits"
$LowercaseLettersRegKey = "LowercaseLetters"
$UppercaseLettersRegKey = "UppercaseLetters"
$SpecialCharactersRegKey = "SpecialCharacters"
$MinimumPINLengthRegKey = "MinimumPINLength"
$MaximumPINLengthRegKey = "MaximumPINLength"
$ExpirationRegKey = "Expiration"
$HistoryRegKey = "History"
```
Next, there are a number of variables that are set to define the different requirements for the backup PIN that needs to be enabled when setting up Windows Hello. I my example below, I only allow numbers and enforce a minimum length of 8. The reason being that the PIN should be different to the user's password. And since the PIN can only be used to log into the device (and thus requires physical access) limiting it to numbers is not a major security risk (it's no different than any PIN on a user's mobile phone).
(These were not setup as parameters as proactive remediations do not allow passing of parameters to the script.)
```powershell
# Set the complexity requirements. Set to 1 to enable the requirement, set to 2 to disable the requirement.
[int]$DigitsRegKeyValue = "1"
[int]$LowercaseLettersRegKeyValue = "2"
[int]$UppercaseLettersRegKeyValue = "2"
[int]$SpecialCharactersRegKeyValue = "2"

# Set the minimum and maximum required PIN length. If you don't want to enfore a maximum length set to 127 which is the longest allowed length.
[int]$MinimumPINLengthRegKeyValue = "8"
[int]$MaximumPINLengthRegKeyValue = "127"

# Set the expiration of the PIN in days up to a maximun of 730. Set to 0 if you do not want the PIN to expire.
[int]$ExpirationRegKeyValue = "0"

# Set the amount of PINs that should be remember and prevented from being re-used. Up to 50 PINs can be remembered. Set to 0 to disable history.
[int]$HistoryRegKeyValue = "0"
```
After setting the requirements, I check if the registry path exists, and if it doesn't I create it.
```powershell
If (!(Test-Path $RegKeyPath)){
    Try {
        # Create the new path
        New-Item $RegKeyPath -ErrorAction Stop -Force > $null
        Write-Host "Registry path created successfully"
    }
    Catch {
        $ErrorMsg = $_.Exception.Message
        Write-host "Error creating registry path: $ErrorMsg"
        Exit 1
    }
}
```
Finally, I perform some validation on the variables set for the complexity of the PIN and then set the registry values. As there are certain limitation in length, and the registry keys need to have a specific value, I want to validate them before applying and potentially causing unexpected behaviour.
```powershell
Try {
    # Check that reg values are within allowed values
    If (($DigitsRegKeyValue -ne 1) -AND ($DigitsRegKeyValue -ne 2)){
        Write-Host "You specified an invalid option for requiring numbers."
        Exit 1
    }
    If (($LowercaseLettersRegKeyValue -ne 1) -AND ($LowercaseLettersRegKeyValue -ne 2)){
        Write-Host "You specified an invalid option for requiring lowercase letters."
        Exit 1
    }
    If (($UppercaseLettersRegKeyValue -ne 1) -AND ($UppercaseLettersRegKeyValue -ne 2)){
        Write-Host "You specified an invalid option for requiring uppercase numbers."
        Exit 1
    }
    If (($SpecialCharactersRegKeyValue -ne 1) -AND ($SpecialCharactersRegKeyValue -ne 2)){
        Write-Host "You specified an invalid option for requiring special characters."
        Exit 1
    }
    If ($MaximumPINLengthRegKeyValue -gt 127) {
        Write-Host "The maximum PIN length needs to be less than 127."
        Exit 1
    }
    If ($MinimumPINLengthRegKeyValue -gt $MaximumPINLengthRegKeyValue){
        Write-Host "The minimum PIN length should be less than the maximum PIN length."
        Exit 1
    }
    If ($ExpirationRegKeyValue -gt 720) {
        Write-Host "The expiration needs to be less than 720 days."
        Exit 1
    }
    If ($HistoryRegKeyValue -gt 50) {
        Write-Host "The number of PINs that are remembered need to be less than 50."
        Exit 1
    }

    # Set the registry values
    Set-ItemProperty -Path $RegKeyPath -Name $DigitsRegKey -Value $DigitsRegKeyValue -Type Dword -ErrorAction Stop -Force
    Set-ItemProperty -Path $RegKeyPath -Name $LowercaseLettersRegKey -Value $LowercaseLettersRegKeyValue -Type Dword -ErrorAction Stop -Force
    Set-ItemProperty -Path $RegKeyPath -Name $UppercaseLettersRegKey -Value $UppercaseLettersRegKeyValue -Type Dword -ErrorAction Stop -Force
    Set-ItemProperty -Path $RegKeyPath -Name $SpecialCharactersRegKey -Value $SpecialCharactersRegKeyValue -Type Dword -ErrorAction Stop -Force
    Set-ItemProperty -Path $RegKeyPath -Name $MinimumPINLengthRegKey -Value $MinimumPINLengthRegKeyValue -Type Dword -ErrorAction Stop -Force
    Set-ItemProperty -Path $RegKeyPath -Name $MaximumPINLengthRegKey -Value $MaximumPINLengthRegKeyValue -Type Dword -ErrorAction Stop -Force
    Set-ItemProperty -Path $RegKeyPath -Name $ExpirationRegKey -Value $ExpirationRegKeyValue -Type Dword -ErrorAction Stop -Force
    Set-ItemProperty -Path $RegKeyPath -Name $HistoryRegKey -Value $HistoryRegKeyValue -Type Dword -ErrorAction Stop -Force

    Write-Host "Registry values set correctly"
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-host "Error setting the registry value: $ErrorMsg"
    Exit 1
}
```

## The Detection Script
The first part of the detection script is the same as the configuration script. I set variable for the registry path and keys, as well as what values are expected for the keys.
```powershell
# Set Variables
$RegKeyPath = "HKLM:\SOFTWARE\Policies\Microsoft\PassportForWork\PINComplexity"
$DigitsRegKey = "Digits"
$LowercaseLettersRegKey = "LowercaseLetters"
$UppercaseLettersRegKey = "UppercaseLetters"
$SpecialCharactersRegKey = "SpecialCharacters"
$MinimumPINLengthRegKey = "MinimumPINLength"
$MaximumPINLengthRegKey = "MaximumPINLength"
$ExpirationRegKey = "Expiration"
$HistoryRegKey = "History"

# Set the expected values for the complexity requirements. 1 if the requirement is enabled, 2 if disabled.
[int]$DigitsRegKeyExpectedValue = "1"
[int]$LowercaseLettersRegKeyExpectedValue = "2"
[int]$UppercaseLettersRegKeyExpectedValue = "2"
[int]$SpecialCharactersRegKeyExpectedValue = "2"

# Set the expected values for minimum and maximum required PIN length. The maximum length needs to be less than 127.
[int]$MinimumPINLengthRegKeyExpectedValue = "8"
[int]$MaximumPINLengthRegKeyExpectedValue = "127"

# Set the expected value of the expiration of the PIN in days up to a maximun of 730. 0 if there is no expiry.
[int]$ExpirationRegKeyExpectedValue = "0"

# Set the expected amount of PINs that should be remember and prevented from being re-used. Up to 50 PINs can be remembered. 0 disables history.
[int]$HistoryRegKeyExpectedValue = "0"

# Get current values of registry keys
$DigitsRegKeyCurrentValue = Get-ItemPropertyValue -Path $RegKeyPath -Name $DigitsRegKey -ErrorAction SilentlyContinue
$LowercaseLettersRegKeyCurrentValue = Get-ItemPropertyValue -Path $RegKeyPath -Name $LowercaseLettersRegKey -ErrorAction SilentlyContinue
$UppercaseLettersRegKeyCurrentValue = Get-ItemPropertyValue -Path $RegKeyPath -Name $UppercaseLettersRegKey -ErrorAction SilentlyContinue
$SpecialCharactersRegKeyCurrentValue = Get-ItemPropertyValue -Path $RegKeyPath -Name $SpecialCharactersRegKey -ErrorAction SilentlyContinue
$MinimumPINLengthRegKeyCurrentValue = Get-ItemPropertyValue -Path $RegKeyPath -Name $MinimumPINLengthRegKey -ErrorAction SilentlyContinue
$MaximumPINLengthRegKeyCurrentValue = Get-ItemPropertyValue -Path $RegKeyPath -Name $MaximumPINLengthRegKey -ErrorAction SilentlyContinue
$ExpirationRegKeyCurrentValue = Get-ItemPropertyValue -Path $RegKeyPath -Name $ExpirationRegKey -ErrorAction SilentlyContinue
$HistoryRegKeyCurrentValue = Get-ItemPropertyValue -Path $RegKeyPath -Name $HistoryRegKey -ErrorAction SilentlyContinue
```

Then I perform the same validation as in the configuration script, but this time it is followed by checking that the registry keys each have the value that was specified earlier in the script. If they don't I through an error that specifies what keys was not correct. This way troubleshooting the issue becomes much easier as you know exactly what section of the script to look at.
```powershell
Try {
    # Check that reg values are within allowed values
    If (($DigitsRegKeyExpectedValue -ne 1) -AND ($DigitsRegKeyExpectedValue -ne 2)){
        Write-Host "You specified an invalid option for requiring numbers."
        Exit 1
    }
    If (($LowercaseLettersRegKeyExpectedValue -ne 1) -AND ($LowercaseLettersRegKeyExpectedValue -ne 2)){
        Write-Host "You specified an invalid option for requiring lowercase letters."
        Exit 1
    }
    If (($UppercaseLettersRegKeyExpectedValue -ne 1) -AND ($UppercaseLettersRegKeyExpectedValue -ne 2)){
        Write-Host "You specified an invalid option for requiring uppercase numbers."
        Exit 1
    }
    If (($SpecialCharactersRegKeyExpectedValue -ne 1) -AND ($SpecialCharactersRegKeyExpectedValue -ne 2)){
        Write-Host "You specified an invalid option for requiring special characters."
        Exit 1
    }
    If ($MaximumPINLengthRegKeyExpectedValue -gt 127) {
        Write-Host "The maximum PIN length needs to be less than 127."
        Exit 1
    }
    If ($MinimumPINLengthRegKeyExpectedValue -gt $MaximumPINLengthRegKeyExpectedValue){
        Write-Host "The minimum PIN length should be less than the maximum PIN length."
        Exit 1
    }
    If ($ExpirationRegKeyExpectedValue -gt 720) {
        Write-Host "The expiration needs to be less than 720 days."
        Exit 1
    }
    If ($HistoryRegKeyExpectedValue -gt 50) {
        Write-Host "The number of PINs that are remembered need to be less than 50."
        Exit 1
    }

    # Perform checks
    If ($DigitsRegKeyCurrentValue -ne $DigitsRegKeyExpectedValue){
        Write-host "Registry key $RegKeyPath\$DigitsRegKey has incorrect value."
        Exit 1
    }
    If ($LowercaseLettersRegKeyCurrentValue -ne $LowercaseLettersRegKeyExpectedValue){
        Write-host "Registry key $RegKeyPath\$LowercaseLettersRegKey has incorrect value."
        Exit 1
    }
    If ($UppercaseLettersRegKeyCurrentValue -ne $UppercaseLettersRegKeyExpectedValue){
        Write-host "Registry key $RegKeyPath\$UppercaseLettersRegKey has incorrect value."
        Exit 1
    }
    If ($SpecialCharactersRegKeyCurrentValue -ne $SpecialCharactersRegKeyExpectedValue){
        Write-host "Registry key $RegKeyPath\$SpecialCharactersRegKey has incorrect value."
        Exit 1
    }
    If ($MinimumPINLengthRegKeyCurrentValue -ne $MinimumPINLengthRegKeyExpectedValue){
        Write-host "Registry key $RegKeyPath\$MinimumPINLengthRegKey has incorrect value."
        Exit 1
    }
    If ($MaximumPINLengthRegKeyCurrentValue -ne $MaximumPINLengthRegKeyExpectedValue){
        Write-host "Registry key $RegKeyPath\$MaximumPINLengthRegKey has incorrect value."
        Exit 1
    }
    If ($ExpirationRegKeyCurrentValue -ne $ExpirationRegKeyExpectedValue){
        Write-host "Registry key $RegKeyPath\$ExpirationRegKey has incorrect value."
        Exit 1
    }
    If ($HistoryRegKeyCurrentValue -ne $HistoryRegKeyExpectedValue){
        Write-host "Registry key $RegKeyPath\$HistoryRegKey has incorrect value."
        Exit 1
    }

    # If all checks pass exit with success code.
    Write-Host "All registry keys have correct values."
    Exit 0
}
Catch {
    $ErrorMsg = $_.Exception.Message
    Write-host "Error $ErrorMsg"
    Exit 1
}
```

And that is all there is to it. As always, if you have any questions or comments please reach out.