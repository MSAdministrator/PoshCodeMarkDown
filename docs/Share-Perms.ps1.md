---
Author: littlegun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3642
Published Date: 2014-09-14t11
Archived Date: 2014-08-14t04
---

# share perms - 

## Description

this script removes all existing permissions and assigns the appropriate domain permissions.  also the �owner� is set to �builtin\administrators�

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $FolderPath = "\\FilerName\ShareName"
 
 
 $rootfolder = Get-ChildItem -Path $FolderPath -recurse 
 foreach ($file in $rootfolder) {
         $file.FullName
         Get-Acl $file.FullName | Format-List
             $acl = Get-Acl $file.FullName 
             $acl.Access | %{$acl.RemoveAccessRule($_)} 
             $acl.SetAccessRuleProtection($True, $False) 
             $Rights = [System.Security.AccessControl.FileSystemRights]::FullControl
             $inherit = [System.Security.AccessControl.FileSystemAccessRule]::ContainerInherit -bor [System.Security.AccessControl.FileSystemAccessRule]::ObjectInherit
             $Propagation = [System.Security.AccessControl.PropagationFlags]::None
             $Access = [System.Security.AccessControl.AccessControlType]::Allow
             $rule = New-Object System.Security.AccessControl.FileSystemAccessRule("DomainName\GroupName",$Rights, $inherit, $Propagation, $Access)
             $acl.AddAccessRule($rule)
             $rule = New-Object System.Security.AccessControl.FileSystemAccessRule("DomainName\GroupName",$Rights, $inherit, $Propagation, $Access)
             $acl.AddAccessRule($rule)
             $acct=New-Object System.Security.Principal.NTAccount("Builtin\Administrators") 
             $acl.SetOwner($acct) 
             Set-Acl $file.FullName $acl 
             Get-Acl $file.FullName  | Format-List
             }
`

