---
Author: richard vantreas
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2999
Published Date: 2012-10-12t15
Archived Date: 2016-08-03t11
---

# convert file encoding - 

## Description

r.vantrease ver 1.0 – the source control i use does not understand the default encoding that powershell_ise saves scripts in (unicode big endian), so i wanted a way to quickly convert my scripts to a friendlier encoding.  i wrote the following little ditty to convert all the powershell scripts and module files to ascii encoding.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 cd c:\windows\temp\common
 
 foreach($File in get-childitem | where-object{($_.Extension -ieq '.PS1') -or ($_.Extension -ieq '.PSM1')})
 {
     $FileName = $File.Name
     $TempFile = "$($File.Name).ASCII"
 
     get-content $FileName | out-file $TempFile -Encoding ASCII 
 
     remove-item $FileName
 
     rename-item $TempFile $FileName
 }
`

