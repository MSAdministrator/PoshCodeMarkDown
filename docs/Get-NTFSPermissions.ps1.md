---
Author: vidrine
Publisher: 
Copyright: 
Email: 
Version: 2013.01.14
Encoding: utf-8
License: cc0
PoshCode ID: 5006
Published Date: 2016-03-21t13
Archived Date: 2016-05-01t21
---

# get-ntfspermissions - 

## Description

specify target host and root directory.  the script will then recursively check for all folders and report on their ntfs permissions.

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
   Author:..Vidrine
   Date:....2013.01.14
 
 .DESCRIPTION
   Thanks to http://jfrmilner.wordpress.com/  
 
   Specify target host and root directory.  The script will then recursively check for all folders and report on their NTFS permissions.
   Output is stored in a custom object, that is then exported to CSV.
 
   Script can easily be scaled to include processing multiple hosts, processing hosts imported from source file, process files instead of just folders, etc...
 #>
 
 $target          = Join-Path -ChildPath $targetDirectory -Path $targetServer
  
 Get-ChildItem -Recurse -Path $target | Where { $_.PSIsContainer } |
 forEach {
     $objPath = $_.FullName
     $coLACL  = Get-Acl -Path $objPath
     forEach ( $objACL in $colACL ) {
         forEach ( $accessRight in $objACL.Access ) {
             $objResults = New-Object �TypeName PSObject
             $objResults | Add-Member �MemberType NoteProperty �Name DirectoryPath      �Value $objPath
             $objResults | Add-Member �MemberType NoteProperty �Name Identity           �Value $accessRight.IdentityReference
             $objResults | Add-Member �MemberType NoteProperty �Name SystemRights       �Value $accessRight.FileSystemRights
             $objResults | Add-Member �MemberType NoteProperty �Name SystemRightsType   �Value $accessRight.AccessControlType
             $objResults | Add-Member -MemberType NoteProperty -Name IsInherited        -Value $accessRight.IsInherited
             $objResults | Add-Member -MemberType NoteProperty -Name InheritanceFlags   -Value $accessRight.InheritanceFlags
             $objResults | Add-Member �MemberType NoteProperty �Name RulesProtected     �Value $objACL.AreAccessRulesProtected
             $arrResults += $objResults
         }
     }
 }
  
 $arrResults | Export-CSV -NoTypeInformation -Path $exportPath
`

