---
Author: vern anderson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6058
Published Date: 2016-10-21t14
Archived Date: 2016-05-17t14
---

# folder inheritance - 

## Description

change the inheritance from parent with powershell

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
    md c:\grandfather\father\son
    md c:\grandmother\mother\daughter
 
 
       $inheritance = get-acl c:\grandmother
       $inheritance.SetAccessRuleProtection($false,$false)
       set-acl c:\grandmother -aclobject $inheritance
       (Get-Acl c:\grandmother).AreAccessRulesProtected
 
 #############################################################
 #############################################################
 
 
       $inheritance = get-acl c:\grandfather
       $inheritance.SetAccessRuleProtection($true,$true)
       set-acl c:\grandfather -aclobject $inheritance
       (Get-Acl c:\grandfather).AreAccessRulesProtected
 
`

