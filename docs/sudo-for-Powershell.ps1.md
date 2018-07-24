---
Author: rbellamy
Publisher: 
Copyright: 
Email: 
Version: 1.4
Encoding: ascii
License: cc0
PoshCode ID: 3057
Published Date: 2012-11-20t13
Archived Date: 2016-10-18t11
---

# sudo for powershell - 

## Description

added hidden window, with export/import-clixml to pull results into current window

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
 #
  
 param (
 		)
 		
 $tempPath = "$env:TEMP:\PoSH-sudo-$PID.xml";
 
 $powershell = (Get-Command powershell).Definition;
 
 $dir = Get-Location;
 
 if([System.IO.File]::Exists("$(Get-Location)\$file")) {
         $file = (Get-ChildItem $file).Fullname;
 }
 
 if ($ps) { 
 
         $psi = New-Object System.Diagnostics.ProcessStartInfo $powershell;
         $psi.WorkingDirectory = Get-Location;
 		
         $sArgs = $file + " " + $arguments;
  
         $psi.Arguments = " -Command Set-Location $dir; $sArgs";
 		
 		$psi.UseShellExecute = $false;
  
         $psi.Verb = "RunAs";
 } else { 
 
         $psi = New-Object System.Diagnostics.ProcessStartInfo $file;
         $psi.Arguments = $arguments;
 		
         $psi.Verb = "RunAs";
 }
 
 $psi.Arguments = $psi.Arguments + " | Export-Clixml -Path `"$tempPath`"";
 $psi.WindowStyle = "Hidden";
 
 [System.Diagnostics.Process]::Start($psi).WaitForExit();
 
 Import-Clixml -Path "$tempPath";
 Remove-Item $tempPath;
`

