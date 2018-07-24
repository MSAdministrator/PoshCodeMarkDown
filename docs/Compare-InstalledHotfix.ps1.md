---
Author: andy schneider
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1296
Published Date: 2009-08-28t07
Archived Date: 2016-12-24t05
---

# compare-installedhotfix - 

## Description

takes two servers and provides the delta of installed hotfixes between them

## Comments



## Usage



## TODO



## function

`compare-installedhotfix`

## Code

`#
 #
 Function Compare-InstalledHotfix {
 param (
 [parameter(Mandatory=$true,Position=0)]
 $server1,
 
 [parameter(Mandatory=$true,Position=1)]
 $server2, 
 
 [parameter(Mandatory=$true,Position=3)]
 [Management.Automation.PSCredential]
 $credential
 )
 
 $server1HotFix = get-hotfix -computer $server1 -Credential $credential | select HotfixId
 $server2HotFix = get-hotfix -computer $server2 -Credential $credential | select HotfixId
 
 $comparedHotfixes = compare-object $server2HotFix $server1HotFix -IncludeEqual
 
 $result = @();
 
 foreach ($c in $comparedHotfixes) {
     $kbinfo = "" | select KB,$server1,$server2
     $kbinfo.KB = $c.InputObject.HotfixId
     switch ($c.SideIndicator)
     {
     "==" {
             write-host -ForegroundColor Green "Both servers have $($c.InputObject.HotfixId)"
             $kbinfo.($server1) = $true
             $kbinfo.($server2) = $true
             $result += $kbinfo
          }
          
     "=>" {
             write-host -ForegroundColor Yellow "$server1 has $($c.InputObject.HotfixId) but $server2 doesn't"
             $kbinfo.($server1) = $true
             $kbinfo.($server2) = $false
             $result += $kbinfo
           }
           
     "<="  {
             write-host -ForegroundColor Magenta "$server2 has $($c.InputObject.HotfixId) but $server1 doesn't"
             $kbinfo.($server1) = $false
             $kbinfo.($server2) = $true
             $result += $kbinfo
           }
    $result
`

