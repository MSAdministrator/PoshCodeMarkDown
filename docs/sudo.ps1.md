---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1804
Published Date: 2010-04-24t13
Archived Date: 2016-05-23t18
---

# sudo - 

## Description

my sudo script runs commands or apps elevated without prompting. but it’s awfully hacky

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 [CmdletBinding()]
 Param(
    [Parameter(ValueFromRemainingArguments=$true)]
    $command=$(Read-Host "You must specify a command")
 )
 
 function global:Sudo {
 ######################################################################
 ######################################################################
 [CmdletBinding()]
 Param(
    [Parameter(ValueFromRemainingArguments=$true)]
    $command=$(Read-Host "You must specify a command")
 )
 
 $SchTasks = ls ([Environment]::GetFolderPath("System")) schtasks.exe
 $SchTasks = $SchTasks.FullName
 $TaskName = '"Elevated PowerShell"'
 
 $AllProfile = Join-Path $(Split-Path $Profile) "Profile.ps1"
 $OutputXml  = Join-Path $(Split-Path $Profile) "SudoOutput.psxml"
 $OutputErr  = Join-Path $(Split-Path $Profile) "SudoOutput.err"
 $command = $($command -join " ") + " 2> $OutputErr | 
 Export-CliXml $OutputXml"
 
 $donecheck = { 
 Get-WinEvent -LogName Microsoft-Windows-TaskScheduler/Operational `
 -FilterXPath "*[System[TimeCreated[timediff(@SystemTime) <= 2500]] 
                and EventData[@Name='TaskSuccessEvent' 
                and Data[@Name='TaskName']='\Elevated PowerShell']]" `
 -ErrorAction "SilentlyContinue"
 }
 
 
 $script = gc $AllProfile
 
 Set-Content $AllProfile @"
 $(($script -notmatch "^\s*#") -join "`n")
 Remove-Item $OutputXml -ErrorAction "SilentlyContinue"
 Write-Host "$Command" -Fore Cyan
 $Command
 "@
 
 $result = &$SchTasks /Run /TN $TaskName
 if($result -notmatch "^SUCCESS") {
    Write-Error $result
 } else {
    while(!(&$donecheck)) { sleep 1 }
 }
 
 $ErrorActionPreference = "SilentlyContinue"
 Import-CliXml $OutputXml
 
 Write-Warning $(@(Get-Content $OutputErr) -join "`n")
 Remove-Item $OutputXml
 Remove-Item $OutputErr
 
 Set-Content $AllProfile $script
 
 }
 
 Sudo $($command -join " ")
`

