---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1996
Published Date: 
Archived Date: 2010-07-21t21
---

#  - 

## Description

search for powercli 4.1 incompatible type references

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Get-PSDrive -PSProvider FileSystem | foreach { $_.Root } | `
    Get-ChildItem -Recurse -Include '*.ps1', '*.psm1', '*.ps1xml' | `
    where { Select-String -Path $_ -SimpleMatch -Pattern `
             'VMware.VimAutomation.Types.', `
             'VMware.VimAutomation.Client20.', `
             '[Datastore]' }
`

