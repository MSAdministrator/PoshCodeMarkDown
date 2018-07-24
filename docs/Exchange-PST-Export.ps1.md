---
Author: st3v3o
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6903
Published Date: 2017-05-23t15
Archived Date: 2017-05-29t01
---

# exchange pst export - 

## Description

export 2007/2010 usermailbox to pst located on file share

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 If (@(Get-PSSnapin -Registered | Where-Object {$_.Name -eq "Microsoft.Exchange.Management.PowerShell.E2010"} ).count -eq 1) {
     If (@(Get-PSSnapin | Where-Object {$_.Name -eq "Microsoft.Exchange.Management.PowerShell.E2010"} ).count -eq 0) {
          Write-Host "Loading Exchange Snapin Please Wait...."; Add-PSSnapin Microsoft.Exchange.Management.PowerShell.E2010}
          } 
 
 If (@(Get-PSSnapin -Registered | Where-Object {$_.Name -eq "Microsoft.Exchange.Management.PowerShell.Admin"} ).count -eq 1){ 
     If (@(Get-PSSnapin | Where-Object {$_.Name -eq "Microsoft.Exchange.Management.PowerShell.Admin"} ).count -eq 0) {
         Write-Host "Loading Exchange Snapin Please Wait...."; Add-PSSnapin Microsoft.Exchange.Management.PowerShell.Admin}
         }
 
 Write-Host "`n`n`t`t Export Mailbox To PST `n`n"
 
 $path = "\\file\share"
 $admin = [Environment]::UserName
 
 $list=Read-Host "Do you want to read from a file? (Y/N)"
 IF ($list -eq "N") { 
 	$user = Read-Host "Enter A User Name"
        Add-MailboxPermission -Identity $user -User $admin -AccessRights  FullAccess
 	   Export-Mailbox $user -PSTFolderPath $path\$user.pst -Confirm:$false
        
 }
 IF ($list -eq "Y") {
     $file = Read-Host "Enter File Path/Name"
 	$users = Get-Content $file
 	Foreach ($user in $users) {
 	   Export-Mailbox $user -PSTFolderPath $path\$user.pst -Confirm:$false
 	}
 }
`

