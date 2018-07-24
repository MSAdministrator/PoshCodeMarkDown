---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5439
Published Date: 2016-09-17t15
Archived Date: 2016-05-24t22
---

# get-ownerreport.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Gets a list of files in the current directory, but with their owner added
 to the resulting objects.
 
 .EXAMPLE
 
 Get-OwnerReport | Format-Table Name,LastWriteTime,Owner
 Retrieves all files in the current directory, and displays the
 Name, LastWriteTime, and Owner
 
 #>
 
 Set-StrictMode -Version Latest
 
 $files = Get-ChildItem
 foreach($file in $files)
 {
     $owner = (Get-Acl $file).Owner
     $file | Add-Member NoteProperty Owner $owner
     $file
 }
`

