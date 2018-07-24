---
Author: hotsnoj
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6106
Published Date: 2016-11-19t22
Archived Date: 2016-03-18t21
---

# vb - 

## Description

makes use of sysinternal’s psexec to get session data from qwinsta for both local and remote computers.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param (
     $ComputerName #(Read-Host -Prompt "Enter a computer name")
 )
 
 if($ComputerName -eq $null) {
     $c = qwinsta 2>&1 | where {$_.gettype().equals([string]) }
 } else {
     $c = psexec "\\$ComputerName" -s qwinsta 2>&1 | where {$_.gettype().equals([string]) }
 }
 $starters = New-Object psobject -Property @{"SessionName" = 0; "Username" = 0; "ID" = 0; "State" = 0; "Type" = 0; "Device" = 0;};
 
 foreach($line in $c) {
      try {
          if($line.trim().substring(0, $line.trim().indexof(" ")) -eq "SESSIONNAME") {
             $starters.Username = $line.indexof("USERNAME");
             $starters.ID = $line.indexof("ID");
             $starters.State = $line.indexof("STATE");
             $starters.Type = $line.indexof("TYPE");
             $starters.Device = $line.indexof("DEVICE");
             continue;
         }
         
         New-Object psobject -Property @{
             "SessionNAme" = $line.trim().substring(0, $line.trim().indexof(" ")).trim(">")
             ;"Username" = $line.Substring($starters.Username, $line.IndexOf(" ", $starters.Username) - $starters.Username)
             ;"ID" = $line.Substring($line.IndexOf(" ", $starters.Username), $starters.ID - $line.IndexOf(" ", $starters.Username) + 2).trim()
             ;"State" = $line.Substring($starters.State, $line.IndexOf(" ", $starters.State)-$starters.State).trim()
             ;"Type" = $line.Substring($starters.Type, $starters.Device - $starters.Type).trim()
             ;"Device" = $line.Substring($starters.Device).trim()
         }
     } catch {
         throw $_;
         #$e = $_;
     }
 }
`

