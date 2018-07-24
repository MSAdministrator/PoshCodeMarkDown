---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3373
Published Date: 
Archived Date: 2012-04-22t08
---

#  - 

## Description

get local security groups and their members from remote computers.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Write-Host "ComputerName in White, then Local Groups in Green, Local Groups' Members in White"
 
 
 
 
 $x = Get-Content ServerList.txt
 
 
 foreach ($j in $x) {
 
 
 
 $strComputer = "$j"
 
 
 
 $computer = [ADSI]("WinNT://" + $strComputer + ",computer")
 $g = $computer.name.ToString()
 $g.ToUpper(); " "
 
 
 
 $group = $computer.psbase.children | where{$_.psbase.schemaclassname -eq "group"}
 $a = @()
 foreach ($member in $group.psbase.syncroot)
 {$a += $member.name } 
 
 
 
 #(Group name printed in green)
 
 foreach ($i in $a) {
 
 $group =[ADSI]"WinNT://$j/$i" 
 Write-Host "$i" -fore green
 $members = @($group.psbase.Invoke("Members")) 
 $members | foreach { Write-Host " ",$_.GetType().InvokeMember("Name", 'GetProperty', $null, $_, $null)}
 }
 
 Write-host " "
 Write-host " "
 Write-host " "
 }
`

