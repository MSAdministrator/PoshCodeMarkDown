---
Author: bsonposh
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 88
Published Date: 2008-12-31t14
Archived Date: 2017-05-22t04
---

# invoke-resetpassword - 

## Description

used to reset a users password

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
     #################################################################
     Param([string]$user,[string]$password,[string]$server,[switch]$auth)
     
     If(!($server)){$server = get-content env:COMPUTERNAME}
     $dePath = "WinNT://$server/$user,user"
     if($auth)
     {
         $lUser = Read-Host "Enter UserName"
         $lPassword = Read-Host "Enter Password"
         $myuser = new-Object System.DirectoryServices.DirectoryEntry($dePath,$lUser,$lPassword)
     }
     else
     {
         $myuser = new-Object System.DirectoryServices.DirectoryEntry($dePath)
     }
     $myuser.psbase.invoke("SetPassword",$password)
     $myuser.psbase.CommitChanges()
     if($?){Write-Host "Reset Password Successful!"}else{Write-Host "Reset Failed"}
`

