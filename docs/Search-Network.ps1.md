---
Author: pnorms
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3063
Published Date: 2012-11-21t09
Archived Date: 2012-12-25t08
---

# search-network - 

## Description

this is my first real powershell script, not sure if anyone has done anything liek this before (i’m sure it has been done) but i figured i�d share it anyway. makes use of a function i found on here by nathan hartley called get-netview.

## Comments



## Usage



## TODO



## function

`get-netview`

## Code

`#
 #
 $timeStamp = Get-Date -UFormat "%m-%d-%Y-%H-%M"
 $systemVars = Gwmi Win32_ComputerSystem -Comp "."
 $userName = $systemVars.UserName
 $compName = $systemVars.Name
 
 
 "Check Service: " + $serviceName > $errorLog; Get-Date >> $errorLog; $compName >> $errorLog; $userName >> $errorLog; "_____________" >> $errorLog;  "" >> $errorLog;
 "Check Service: " + $serviceName > $fullLog; Get-Date >> $fullLog; $compName >> $fullLog; $userName >> $fullLog; "_____________" >> $fullLog;  "" >> $fullLog;
 
 function Get-NetView {
 	switch -regex (NET.EXE VIEW) { "^\\\\(?<Name>\S+)\s+" {$matches.Name}}
     }
 
 function Process-PCs ($currentName) {
     $olStatus = Ping-Address $currentName
     If ($olStatus -eq "True") {
     Check-Service $currentName
     }
     Write-Host " "
     }
 
 function Ping-Address ($pingAddress) {
     $ping = new-object system.net.networkinformation.ping
     $pingReply = $ping.send($pingAddress)
         If ($pingReply.status -eq "Success") {
             Return "True"
             }
         Else {
             Return "False"
             }
     }
     
 function Check-Service ($currentName) {
     $currentService = Get-Service -ComputerName $currentName -Name $serviceName -ErrorAction SilentlyContinue
     If ($currentService.Status -eq $Null){
         $currentServiceStatus = "Not Installed"
         $currentName >> $errorLog
         }
     ElseIf ($currentService.Status -eq "Running"){
         $currentServiceStatus = "Running"
         }
     ElseIf ($currentService.Status -eq "Stopped"){
         $currentServiceStatus = "Stopped"
         }
     Else {
         $currentServiceStatus = "Unknown"
         }
         
         Write-Host $serviceName " is " $currentServiceStatus " on " $currentName
         $serviceName + " was " + $currentServiceStatus + " on " + $currentName >> $fullLog
     If ($currentService.Status -eq "Stopped"){
         Write-Host "Service was stoppped, trying to start it . . ."
         $currentService | Start-Service -ErrorAction SilentlyContinue
         If ($currentService.Status -eq "Running"){
             "   Service Successfully Started" >> $fullLog
             }
         Else {
             "   Service Could Not Be Started" >> $fullLog
             }
        }
     }
 
 
 cls
 Get-NetView | %{Process-PCs $_}
`

