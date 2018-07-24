---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5419
Published Date: 2015-09-12t05
Archived Date: 2015-01-31t20
---

# search-poshcodescript - 

## Description

i found this script very useful. enjoy!

## Comments



## Usage



## TODO



## function

`search-poshcodescript`

## Code

`#
 #
 function Search-PoshcodeScript {
   <#
     .EXAMPLE
         PS C:\> Search-PoshcodeScript "logon users"
     .NOTES
         Author: greg zakharov
   #>
   param(
     [Parameter(Mandatory=$true)]
     [String]$ScriptName
   )
   
   &(([Regex]"(?<=`")(.*)(?=`"\s)").Match(
     (cmd /c ftype (cmd /c assoc .html).Split('=')[1])
   ).Value) ('http://poshcode.org/?lang=&q=' + $ScriptName -replace '\s', '+')
 }
`

