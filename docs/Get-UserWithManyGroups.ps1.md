---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1385
Published Date: 
Archived Date: 2010-07-17t01
---

# get-userwithmanygroups - 

## Description

lists active directory user accounts which are members of too many groups, and can thus cause token bloat issues

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 $limit = 75
 Get-QADUser -SizeLimit 0 -DontUseDefaultIncludedProperties |
   ForEach-Object {
     $groups = Get-QADGroup -ContainsIndirectMember $_.DN -SizeLimit $limit `
       -DontUseDefaultIncludedProperties -WarningAction SilentlyContinue
     if ($groups.Count -ge $limit) { $_ }
   }
`

