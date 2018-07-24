---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5424
Published Date: 
Archived Date: 2015-01-14t05
---

#  - 

## Description

here is full description provided by the customer

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
         .SYNOPSIS
         This script cleans some directories in Users Profiles folder on a terminal server.
                 The script need some modifications, pleasee .NOTES section.
                
         .DESCRIPTION
         This script cleans the following directories in all Users Profiles on the server:
                         \AppData\Local\Temp
                         \AppData\Local\Microsoft\Windows\Temporary Internet Files
                         \AppData\Local\Google\Chrome\User Data\Default\Cache
                         \AppData\Local\Google\Chrome\User Data\Default\Media Cache
                         \UPM_Profile\AppData\Local\MigWiz
                
                 The script logs its work to Windows Log -> Application. You can use standard Log Viewer (
                 My Computer -> Rigth Click -> Manage -> Event Viewer -> Windows Logs -> Application) to view script's logs.
                
         .NOTES
                 The path to Users Profiles directory is hardcoded in the script and should be modified!
                 Please modify $ScriptName variable according your needs. You can find this variable in line 37.
                
                 The script need administrative rights to perform correctly.
                
                 AUTOR: Lobanov Vitaliy (hdablin)
                 CREATED: 14-sep-2014
                 VERSION: 001
        
         .LINK  
         https://www.odesk.com/applications/280206832
 #>
  
 Clear-Host | Out-Null
  
 <#
         !!!YOU MUST MODIFY THIS VARIABLE!!!
         Put here your actual path such as "C:\Users\" or something else.
 #>     
 $UserProfilesPath = "C:\Users\Vitaliy_Lobanov\Documents\_Develop\Scripts\Clean user profiles directory\Test folder"
  
 $ScriptName = "Clean Windows Users Profiles Script"
  
 $SubDirectoriesToClean =        (      
                                                                 "\AppData\Local\Temp",
                                                                 "\AppData\Local\Microsoft\Windows\Temporary Internet Files",
                                                                 "\AppData\Local\Google\Chrome\User Data\Default\Cache",
                                                                 "\AppData\Local\Google\Chrome\User Data\Default\Media Cache",
                                                                 "\AppData\Local\MigWiz"
                                                         )
  
 New-EventLog �LogName Application �Source $ScriptName -ErrorAction SilentlyContinue | Out-Null
  
 If ((Test-Path $UserProfilesPath) -and (-not ($UserProfilesPath -eq $null)))
  
         $UserProfiles = Get-ChildItem $UserProfilesPath
  
         Foreach ($UserProfile in $UserProfiles)
         {
                 $UserProfilePath = $UserProfile.FullName
                         the root of Users Profile folder, the script ignore them #>            
                 If (Test-Path $UserProfilePath -PathType Container)
                 {
                         Foreach ($Subdir in $SubDirectoriesToClean)
                         {
                                 $FolderToClean = Join-Path -Path $UserProfilePath -ChildPath $Subdir
                                 Write-Host "Cleaning `"$FolderToClean`" folder"
                                
                                 Get-ChildItem -Path $FolderToClean -Recurse | Remove-Item -force -recurse -ErrorAction SilentlyContinue
                
                                 {
                                         Write-EventLog �LogName Application �Source $ScriptName �EntryType Information �EventID 1 �Message �The folder `"$FolderToClean`" cleaned successfully� | Out-Null
                                         Write-Host �The folder `"$FolderToClean`" cleaned successfully�
                                 {
                                         Write-EventLog �LogName Application �Source $ScriptName �EntryType Error �EventID 2 �Message �Cannot clean `"$FolderToClean`". Probably some files or folders are in use.� | Out-Null
                                         Write-Host �Cannot clean `"$FolderToClean`". Probably some files or folders are in use.�
                                 }
                
                         }
                        
                 }
         }      
 } else
         Write-EventLog �LogName Application �Source $ScriptName �EntryType Error �EventID 2 �Message �Cannot find `"$UserProfilesPath`" directoty.� | Out-Null
         Write-Host �Cannot find `"$UserProfilesPath`" directoty.�
 }
`

