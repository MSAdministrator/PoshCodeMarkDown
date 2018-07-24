---
Author: http
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5955
Published Date: 2015-07-31t09
Archived Date: 2015-08-02t00
---

# dhcp backup by andre - 

## Description

dhcp_bkp.ps1 is a script i came up with to backup dhcp servers. i deploy this script using sccm 2012 r2 with a help of two batch files.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 REM Copy script files from deployment starting point
 MD "%ProgramFiles%\DHCP_Backup_Script"
 COPY /Y "%~dp0\dhcp_bkp.ps1" "%ProgramFiles%\DHCP_Backup_Script\dhcp_bkp.ps1"
 COPY /Y "%~dp0\7z.exe" "%ProgramFiles%\DHCP_Backup_Script\7z.exe"
 COPY /Y "%~dp0\7z.dll" "%ProgramFiles%\DHCP_Backup_Script\7z.dll"
 
 REM Create Scheduled Task (Compatible with Windows 2003!)
 schtasks /Create /TN "DHCP Backup" /TR "\"%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe\" -noprofile -executionpolicy Unrestricted -File \"%ProgramFiles%\DHCP_Backup_Script\dhcp_bkp.ps1\"" /SC weekly /D MON,TUE,WED,THU,FRI /ST 17:00 /F /RU <USERNAME> /RP <PASSWORD>
 
 REM Remove script files
 RD  /S /Q "%ProgramFiles%\DHCP_Backup_Script"
 RD  /S /Q "%ProgramFiles(x86)%\DHCP_Backup_Script"
 REM Remove Scheduled Task
 schtasks.exe /Delete /TN "DHCP Backup" /F
 
 
 if (-not (test-path "$env:ProgramFiles\DHCP_Backup_Script\7z.exe")) {throw "$env:ProgramFiles\DHCP_Backup_Script\7z.exe needed"} 
 set-alias sz "$env:ProgramFiles\DHCP_Backup_Script\7z.exe"
 
 $zipLimit = (Get-Date).AddDays(-1)
 sz a -Y -tzip $zipFilename $filesToBkp -SSW
 
 Get-ChildItem $zipDest -Recurse | ? {
   -not $_.PSIsContainer -and $_.CreationTime -lt $zipLimit
 } | Remove-Item -Force
`

