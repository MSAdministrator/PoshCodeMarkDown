---
Author: tony sathre
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6064
Published Date: 2016-10-22t19
Archived Date: 2016-10-30t02
---

# migrate-userprinters - 

## Description

used for migrating a users network printer connections from one server to another. useful when migrating your printers to a new print server and need don’t want to make your users remap their printers manually. if deployed with sccm make sure to set the package to run as the user and not system.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 param (
     [Parameter(Mandatory=$true)]
         [string]$OldPrintServer,
     [Parameter(Mandatory=$true)]
         [string]$NewPrintServer
 )
 
 #$ErrorActionPreference = 'SilentlyContinue'
 $Printers = Get-WmiObject -Class Win32_Printer | where { $_.Network -eq $true }
 
 foreach ($Printer in $Printers) {
     if ($Printer.SystemName -eq "\\$OldPrintServer") {            
         $ShareName = $Printer.ShareName
         $Name = $Printer.Name -replace $OldPrintServer, $NewPrintServer
         (New-Object -ComObject WScript.Network).AddWindowsPrinterConnection("\\$NewPrintServer\$ShareName") | Out-Null
 
         if ($Printer.Default -eq $true) {
             (Get-WmiObject -Class Win32_Printer | where { $_.Name -eq $Name}).SetDefaultPrinter() | Out-Null
         }
 
         $Printer.Delete() | Out-Null
     }
 }
`

