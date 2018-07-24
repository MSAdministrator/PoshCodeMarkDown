---
Author: jan egil ring
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2764
Published Date: 2011-07-03t09
Archived Date: 2011-07-06t05
---

# add-printerdriver - 

## Description

add-printerdriver is a powershell function to install all printer drivers from a specified print server. the function is primarily targeted at remote desktop services (formerly terminal services) session host servers.

## Comments



## Usage



## TODO



## function

`add-printerdriver`

## Code

`#
 #
 function Add-PrinterDriver {
 <#
 .SYNOPSIS
 Adds printer drivers to the local computer from a specified print server.
 .DESCRIPTION
  Adds printer drivers to the local computer from a specified print server. The function collects all shared printer objects from the specified print server and installs them on the local computer if not already installed. One mandatory parameter: PrintServer
 .PARAMETER PrintServer
 The name of the print server to add printer drivers from
 .PARAMETER Clean
 Switch parameter which deletes all network printer connections for the current user.
 .EXAMPLE
 Add-PrinterDriver -PrintServer srv01.domain.local
 Add printer drivers from the specified print server
 .EXAMPLE
 Add-PrinterDriver -PrintServer srv01.domain.local -Clean
 Add printer drivers from the specified print server, then removes all network printer connections for the current user.
 .EXAMPLE
 Add-PrinterDriver -PrintServer srv01.domain.local -Verbose
 Add printer drivers from the specified print server with the -Verbose switch parameter
 .NOTES
 AUTHOR:    Jan Egil Ring
 BLOG:      http://blog.powershell.no
 LASTEDIT:  03.07.2011
 You have a royalty-free right to use, modify, reproduce, and
 distribute this script file in any way you find useful, provided that
 you agree that the creator, owner above has no warranty, obligations,
 or liability for such use.
 #>
  [CmdletBinding()]
      	Param(
               [Parameter(Mandatory=$true)]
               [string] $PrintServer,
 			  [switch] $Clean
              )
 
 $allprinters = @(Get-WmiObject win32_printer -ComputerName $PrintServer -Filter 'shared=true')
 $drivers = @($allprinters | Select-Object drivername -Unique)
 $printers = @()
 foreach ($item in $drivers){
 $printers += @($allprinters | Where-Object {$_.drivername -eq $item.drivername})[0]
 }
 
 $localdrivers = @()
 foreach ($driver in (Get-WmiObject Win32_PrinterDriver)){
 $localdrivers += @(($driver.name -split ",")[0])
 }
 
 $CurrentPrinter = 1
 
 foreach ($printer in $printers) {
 
 Write-Progress -Activity "Installing printers..." -Status "Current printer: $($printer.name)" -Id 1 -PercentComplete (($CurrentPrinter/$printers.count) * 100)
 
 $outputobject = @{}
 $outputobject.drivername = $printer.drivername
 
 $locallyinstalled = $localdrivers | Where-Object {$_ -eq $printer.drivername}
 if (-not $locallyinstalled) {
 Write-Verbose "$($printer.drivername) is not installed locally"
 $AddPrinterConnection = Invoke-WmiMethod -Path Win32_Printer -Name AddPrinterConnection -ArgumentList ([string]::Concat('\\', $printer.__SERVER, '\', $printer.ShareName)) -EnableAllPrivileges
 $outputobject.returncode = $AddPrinterConnection.ReturnValue
 }
 else
 {
 Write-Verbose "$($printer.drivername) is already installed locally"
 $outputobject.returncode = "Already installed"
 }
 
 New-Object -TypeName PSObject -Property $outputobject
 
 $CurrentPrinter ++
 
 }
 
 if ($clean) {
 $printers = Get-WmiObject Win32_Printer -EnableAllPrivileges -Filter network=true
 if ($printers) {
 foreach ($printer in $printers) {
 $printer.Delete()
 }
 }
 }
 }
`

