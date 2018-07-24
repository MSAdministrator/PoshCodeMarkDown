---
Author: littlegun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3641
Published Date: 2012-09-14t11
Archived Date: 2012-09-21t03
---

# home directory perms - 

## Description

completely redoes home directory perms

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 $FolderPath = "\\site filer\userdata$\"
 
 
 $rootfolder = Get-ChildItem -Path $FolderPath -recurse 
 foreach ($file in $rootfolder) {
         $file.FullName
         Get-Acl $file.FullName | Format-List
             $acl = Get-Acl $file.FullName 
             $acl.Access | %{$acl.RemoveAccessRule($_)} 
             $acl.SetAccessRuleProtection($False, $True) 
             $Rights = [System.Security.AccessControl.FileSystemRights]::FullControl
             $inherit = [System.Security.AccessControl.FileSystemAccessRule]::ContainerInherit -bor [System.Security.AccessControl.FileSystemAccessRule]::ObjectInherit
             $Propagation = [System.Security.AccessControl.PropagationFlags]::None
             $Access = [System.Security.AccessControl.AccessControlType]::Allow
             $acct=New-Object System.Security.Principal.NTAccount("Builtin\Administrators") 
             $acl.SetOwner($acct) 
             Set-Acl $file.FullName $acl 
             Get-Acl $file.FullName  | Format-List
             
         }
 
 Write-Host "##########################################" -ForegroundColor Green
 Write-Host "##########################################" -ForegroundColor Green
 
 $rootfolder = Get-ChildItem -Path $FolderPath -recurse 
 foreach ($userfolder in $rootfolder) {
         $userfolder.FullName
         If (get-user "DomainName\$userfolder") {
             Get-Acl $userfolder.FullName | Format-List
             $acl = Get-Acl $userfolder.FullName
             $rule = New-Object System.Security.AccessControl.FileSystemAccessRule($userfolder.Name,"Modify", "ContainerInherit, ObjectInherit", "None", "Allow")
             $acl.AddAccessRule($rule)
             Set-Acl $userfolder.FullName $acl
             Get-Acl $userfolder.FullName  | Format-List
             }
        
 }
`

