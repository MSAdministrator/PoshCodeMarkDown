---
Author: aphexenator
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6662
Published Date: 2017-01-03t19
Archived Date: 2017-01-09t07
---

# rebuild windows indexing - 

## Description

rebuilds windows indexing on a remote pc

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $PCNAME = Read-Host -Prompt "Which is the user's PC?"
 $PING = (test-connection $PCNAME -count 1 -quiet)
     If ( $PING -eq $false)
         {
         Write-Host "Connection Failed"
         return
         }
 
 
 (Get-Service -Name wsearch -ComputerName $PCNAME).stop()
 Write-Host "Windows search service has stopped"
 
 start-sleep -seconds 5
 C:\Windows\psexec -s \\$PCNAME cmd /c del "%ProgramData%\Microsoft\Search\Data\Applications\Windows\Windows.edb"
 
 
 (Get-Service -Name wsearch -ComputerName $PCNAME).start()
 Write-Host "Windows search service has started"
 
 C:\Windows\PsExec.exe -s \\$PCNAME msg * /w "The windows indexing service has restarted, the database can take up to 7 days to rebuild."
 Write-Host "The windows indexing service has restarted, the database can take up to 7 days to rebuild."
`

