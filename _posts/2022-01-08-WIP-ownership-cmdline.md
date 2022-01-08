---
layout: post
read_time: true
show_date: true
title:  Marking WIP files as personal through the command-line
date:   2022-01-08 10:00:00 +1000
description: How to set Windows information protection ownership to personal from the command-line.
img: posts/2022-01-08-WIP-ownership-cmdline/hero.png
tags: [powershell]
author: Peter Dodemont
---
A little while ago I had to deploy SQL Express to a number of devices that needed it for an in-house developed application. The application was previously using SQL LocalDB, but for reasons I will not go into here it needed to be moved over to SQL Express. I started writing a script to do the migration of the databases from where LocalDB stored the database files (in the user's appdata) to a defined location (let's say c:\Database). This all went fine and testing on my device as well as some test users all came back working. With everything tested and working, I started deploying it to general users.

## The Problem
This is where we started running into issues with SQL Express not being able to access some of the files. After a lot of investigation by some of the development team and myself we concluded that Windows Information Protection (WIP) was sometimes preventing SQL Express from accessing the files, even when SQL Express was marked as a corporate or an exempt app.

## Finding The Solution
The solution to this problem seems simple, just change the WIP ownership to be personal and the problem goes away. This had to be done in a script as the application was being rolled out to a significant number of users. The problem is that almost no information seems to be available on how to change the WIP ownership of a file through the command-line or from within a script.
After quite a few hours of fruitless searching for a solution, I decided the look into what would be required to write a little application to be able to achieve this. I had commitments that I needed to get to, so I had to stop and pick this up the next day. Unbeknownst to me at the time one of the in-house developers was actually doing the exact same thing and came to the same conclusion of writing an application for this. He ended up further down the path as me and then came across the solution.

## The Solution
The solution was a Microsoft application that comes with every single Windows install called cipher.exe. This application has several different functionalities, but the important one for us is that it can decrypt files encrypted with EFS (Encrypting File System). WIP uses EFS to mark the ownership of the files as corporate or personal. Personal files just don't have any ownership information defined, which will come in handy shortly. While cipher.exe cannot be used to mark files as corporate (at least as far as I can tell), when ask cipher to decrypt a WIP protected file it will also strip all the ownership information from the file. This then means the file is no longer protected by WIP as it is now considered a personal file.

## The Command
There are a number of switches that can be used with cipher, but the one we are after is "/d" to use the decrypt mode and "/s" to specify a folder and decrypt all the files and sub-folders in the specified folder.
The actual command ends up being really simple and since the script ran under from Intune (so under the SYSTEM account) we also didn't have any permissions issues (as you need to be able to read and modify the file to be able to decrypt it).
Below is the command I ended up using and how I incorporated it into my PowerShell script.

Command-line:
```
Cipher /d /s:C:\Database
```
PowerShell (I had to do it this way as otherwise the arguments didn't pass correctly):
```powershell
$NewDBPath = "$env:SystemDrive\Database"

# Remove enterprise context of new db files
$CipherRun = Cipher /d /s:"$NewDBPath" 2>&1
If ($LASTEXITCODE -ne 0){
    Throw "File Corporate Ownership Error: $CipherRun"
} 
```

As always if you have any questions, please reach out.