---
Author: anonymous
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4000
Published Date: 2015-03-06t21
Archived Date: 2015-05-05t01
---

# update web.config - 

## Description

the beauty of iis web.config files is they are just text files. this function can be used to update values such as computer names in connection strings or any other matched string.  note that the replace function is case sensitive.

## Comments



## Usage



## TODO



## function

`update-wccontents`

## Code

`#
 #
 Function Update-WCContents($File,$SearchString,$NewValue){
     $Contents = Get-Content -Path $File
     $Contents | %{$_.Replace($SearchString,$NewValue)} | Set-Content $File    
`

