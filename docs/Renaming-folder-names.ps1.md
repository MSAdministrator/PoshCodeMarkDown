---
Author: srinu
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6088
Published Date: 2015-11-12t17
Archived Date: 2015-11-13t23
---

# renaming folder names - 

## Description

i just want to rename the folder names in particular path where folder name starts with keep all characters by dropping if “12_” before the file name. i am able to rename it but not able to get log file for the following script.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $Folders = Get-ChildItem 'E:\Root Folder\12_*'
 
 $collection = $()
 Foreach ($Folder in $Folders)
 
 {
 
 $status = @{ "OldFolderName" = $Folder.Name; "NewFolderName" = $Folder.Name.Substring(3); "TimeStamp" = (Get-Date -f yyyy:MM:dd-hh:mm:ss) }
 
 $NewFolderName = $Folder.Name.Substring(3); 
 Rename-Item -Path $Folder -NewName $NewFolderName
 
 New-Object -TypeName PSObject -Property $Status 
   
 }
 
 $collection | Out-File D:\$((Get-Date).ToString('yyyyMMdd'))_Renaming_Folder_Names_script_Log.txt -Append
`

