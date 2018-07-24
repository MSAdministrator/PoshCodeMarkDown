---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5121
Published Date: 2015-04-28t05
Archived Date: 2015-01-31t20
---

# drivers list - 

## Description

fixed

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 gp -ea 0 HKLM:\SYSTEM\CurrentControlSet\Services\* | ? {
   $_.Type -eq 1 -and $_.ImagePath -ne $null
 } | select @{
   N='Name';E={$_.PSChildName}
 }, @{
   N='Path';E={
     $$ = $_.ImagePath.ToLower()
     $pat = Split-Path -Leaf ($dir = [Environment]::SystemDirectory)
     if ($$.StartsWith('\')) {
       $$ = $$.Substring((($i = $$.IndexOf('\', 2)) + 1), ($$.Length - $i - 1))
       if (Test-Path $$) { return $$ }
     }
     $script:itm = [Regex]::Replace($$, $pat, $dir)
     $itm
   }
 }, @{
   N='Description';E={
     $script:des = (gci $itm).VersionInfo
     $des.FileDescription
   }
 }, @{
   N='Publisher';E={$des.CompanyName}
 }, @{
   N='Version';E={$des.ProductVersion}
 }, @{
   N='Error Control';E={'0x{0:X8}' -f $_.ErrorControl}
 }, @{
   N='Launch Type';E={
     switch ($_.Start) {
       4 { 'Disabled' }
     }
   }
 } | Out-GridView -Title Drivers
`

