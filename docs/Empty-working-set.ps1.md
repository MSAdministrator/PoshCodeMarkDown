---
Author: amirul
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6808
Published Date: 2017-03-20t23
Archived Date: 2017-03-25t17
---

# empty working set - 

## Description

pipe filter which empties working set for any received system.diagnostics.process object

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 add-type -Namespace Win32 -Name Psapi -MemberDefinition @"
 [DllImport("psapi", SetLastError=true)]
 public static extern bool EmptyWorkingSet(IntPtr hProcess);    
 "@
  
 filter Reset-WorkingSet {
     [Win32.Psapi]::EmptyWorkingSet($_.Handle)
 }
  
 sal trim Reset-WorkingSet
`

