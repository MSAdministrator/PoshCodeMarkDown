---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3408
Published Date: 
Archived Date: 2012-05-13t17
---

#  - 

## Description

get-casconnectlogentries.ps1

## Comments



## Usage



## TODO



## function

`get-clientversionconnectlogs`

## Code

`#
 #
 #------------------------------------------------------------------------------
 #
 #
 #------------------------------------------------------------------------------
  
 
 Function Get-ClientVersionConnectLogs () {
      Param(
            $LogDate=$Null
       )
 
      If ($LogDate) {
            $Script:CAS | %{gc ( '\\' + $_.Name + '\c$\Program Files\Microsoft\Exchange Server\V14\Logging\RPC Client Access\RCA_'+$T+'-*.LOG') |?{$_-match"OUTLOOK.EXE"-and$_-match",Connect,"}}
       }
 }
 
 
 #
 $Script:CasLogUNCDir   ='\\<server>\<drv>\Data\CASConnectLogs'
 
 $Script:CAS = Get-ExchangeServer | ?{ $_.IsClientAccessServer -eq $true -and $_.AdminDisplayVersion -match "^Version 14" }
 
  
 1..14 | % {
 
      $T=Get-Date ((Get-Date).adddays(($_) * -1)) -Format 'yyyyMMdd'
      $Script:CASLogFile           = $('OLConnect-' + $T + '.txt')
 
      $FileName = Join-Path -Path $Script:CasLogUNCDir -ChildPath $Script:CASLogFile
 
      if(-not (Test-Path $FileName)) {
 
            Write-output "Working: " $T
 
            Get-ClientVersionConnectLogs $T | Out-File $FileName
 
       }
 }
 
  
 
 
 gci $Script:CasLogUNCDir | ? {$_.LastWritetime -lt ((get-date).adddays(-30))} | Remove-Item | Out-Null
`

