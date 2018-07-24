---
Author: nigel boulton
Publisher: 
Copyright: 
Email: 
Version: 1.00
Encoding: ascii
License: cc0
PoshCode ID: 664
Published Date: 
Archived Date: 2009-01-05t18
---

# execute in powershell - 

## Description

adds ‘execute in powershell’ options to .ps1 files’ context menu so that you can easily run scripts from windows explorer

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #===================================================================================
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #===================================================================================
 
 $PoSH = Get-Process | Where-Object {$_.Name -eq "PowerShell"} | Select-Object -First 1
 If ($PoSH -eq $null) {
 	$PoSHPath = Read-Host "Please enter the full path to PowerShell.exe, or cancel and run this script in a PowerShell console session"
 	If ($PoSHPath -eq $Null) {exit 1}
 	}
 Else
 	{
 	$PoSHPath = $PoSH.Path
 }
 
 If ($PoSHPath.Substring(0,1) -ne "`"") {$PoSHPath = "`"" + $PoSHPath}
 If ($PoSHPath.Substring($PoSHPath.Length - 1,1) -ne "`"") {$PoSHPath += "`""}
 
 $PS1Class = (Get-ItemProperty -Path HKLM:SOFTWARE\Classes\.ps1)."(Default)"
 
 $RegData = @{"HKLM:Software\Classes\$PS1Class\shell\Execute in PowerShell\command" = $PoSHPath + " &'%1'";`
 			 "HKLM:Software\Classes\$PS1Class\shell\Execute in PowerShell (-NoExit)\command" = $PoSHPath + " -NoExit &'%1'"}
 
 If ((Test-Path "HKLM:Software\Classes\$PS1Class\shell\open\command") -eq $False) {$RegData.Add("HKLM:Software\Classes\$PS1Class\shell\open\command", "notepad.exe `"%1`"")}
 
 $RegData.GetEnumerator() | ForEach-Object {
 	If ((Test-Path $_.Name) -eq $False) {New-Item -Path $_.Name -Force | Out-Null}
 	Set-ItemProperty -Path $_.Name -Name "(Default)" -Value $_.Value
 }
 Write-Output "Processing complete"
`

