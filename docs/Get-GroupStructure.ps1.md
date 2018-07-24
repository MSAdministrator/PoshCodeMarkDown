---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5538
Published Date: 2016-10-24t11
Archived Date: 2016-05-17t16
---

# get-groupstructure - 

## Description

this simple function exports the structure of nested groups in a similar way as folder and file structures are usually presented.

## Comments



## Usage



## TODO



## function

`get-groupstructure`

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 function Get-GroupStructure
 {
     <#
     .SYNOPSIS
     This cmdlets exports the structure of nested groups and users.
 
     .DESCRIPTION
     This cmdlets exports the structure of nested groups and users, in a simliar way
     as file structures are presented.
 
     It requires the Active Directory module to run.
 
     .EXAMPLE
     Get-GroupStructure -GroupName "Domain Admins"
 
     .PARAMETER GroupName
     Specify the name of the "root group".
 
     .PARAMETER GroupPath
     Set the "start level" of the returned string. Mostly used internally, you can safely ignore this.
 
     #>
 
     param ([string] $GroupPath = '',
            [string] $GroupName)
 
     $GroupMembers = @()
     $GroupMembers += Get-ADGroupMember $GroupName | Sort-Object objectClass -Descending
 
     $LoopCount = @($GroupPath -split " \\ " | Where-Object { $_ -eq $GroupName })
 
     if ($LoopCount.Count -ge 2) {
         Write-Error "Nested group loop detected. Group: $GroupName"
         return;
     }
 
     if ($GroupPath -eq '') {
         $GroupPath = "$GroupName \ "
     }
 
     if ($GroupMembers.Count -eq 0) {
         Write-Output $GroupPath
     }
 
     foreach($GroupMember in $GroupMembers) {
         
         Remove-Variable DrilledDownGroupPath, UserPath -ErrorAction SilentlyContinue
 
         if ($GroupMember.objectClass -eq 'group') {
             $DrilledDownGroupPath = $GroupPath + "$($GroupMember.name) \ "
             Get-GroupStructure -GroupPath $DrilledDownGroupPath -GroupName $GroupMember.name
         }
         else {
             $UserPath = "$GroupPath$($GroupMember.Name) ($($GroupMember.SamAccountName))"
             Write-Output $UserPath
         }
     }
 }
`

