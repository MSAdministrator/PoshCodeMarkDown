---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 732
Published Date: 
Archived Date: 2009-03-12t06
---

# regquery - 

## Description

parses output of registry utility reg query for the pattern on the specified computer. useful for finding installed s/w since the wmi provider for installed software is not always reliable.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param ($computer,$regkey,$pattern)
 
 $p = $($regkey -replace "HKLM","HKEY_LOCAL_MACHINE") -replace "\\","\\"
 $p += "(?<pattern>.*$pattern.*)"
 $matchArray = @()
 
 REG QUERY \\$computer\$regKey | foreach {  if ($_ -match $p) { $matchArray += $matches.pattern }}
 if ($matchArray.length -gt 0)
 { $matchArray | foreach {new-object psobject |
             add-member -pass NoteProperty computer $computer |
             add-member -pass NoteProperty pattern $_}}
 else {new-object psobject |
             add-member -pass NoteProperty computer $computer |
             add-member -pass NoteProperty pattern $null}
`

