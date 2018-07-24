---
Author: kris cieslak
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1800
Published Date: 2011-04-21t13
Archived Date: 2011-08-10t11
---

# enable/disable nic, xp+ - 

## Description

enabling/disabling network adapter. works on windows xp and higher.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 PARAM ($ifname = $(throw "Specifiy interface name"),$state = "")
 trap [Exception] {
     Write-Host 'Ups! Something wrong.' -ForegroundColor Red
 	continue;
 }
 
 $UpStateLabel = 'En&able';
 $DownStateLabel = 'Disa&ble';
 
 $st = 0;
 if ($state.length -gt 0) {
   switch ($state.ToLower()) {
        'up' { $st = 1 }
 	   'down' {$st = 0 }
   }
 } else {
   $c =(gwmi Win32_NetworkAdapter | ? { $_.NetConnectionID -eq $ifname }).ConfigManagerErrorCode;
   if ($c -eq 22) { $st = 1 } else { $st = 0 }
 }
 
 if ($st -eq 1) {
     $StateLabel = $UpStateLabel;
 } else {
     $StateLabel = $DownStateLabel;
 }
 
 if ([int](([regex]('\d{1,3}')).match((gwmi win32_OperatingSystem).Version).ToString()) -le 5) {
     $shell = New-Object -comObject Shell.Application;
     $test=(($shell.NameSpace(3).Items() | 
 	    ? { $_.Path -like '*7007ACC7-3202-11D1-AAD2-00805FC1270E*'}).GetFolder.Items() |
 		? { $_.Name -eq $ifname }).Verbs() | ? { $_.Name -eq $StateLabel }
 	if ($test -ne $null) { 
 	   ($test).DoIt() 
 	}
 } else {
      if ($st -eq 1) {
         (gwmi Win32_NetworkAdapter | ? { $_.NetConnectionID -eq $ifname } ).Enable() | Out-Null
 	 } else {
 	    (gwmi Win32_NetworkAdapter | ? { $_.NetConnectionID -eq $ifname } ).Disable() | Out-Null
 	 }
 }
`

