---
Author: sdfsdfsdfsdfsdf
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6440
Published Date: 2016-07-01t20
Archived Date: 2016-08-14t13
---

# send-popup - 

## Description

send a popup message to a user on a remote computer.

## Comments



## Usage



## TODO



## function

`send-popup`

## Code

`#
 #
 function Send-Popup {
 
     param ($Computername,$Message)
 
     if (Test-Connection -ComputerName $Computername -Count 1 -Quiet){
         Invoke-Command -ComputerName $Computername -ScriptBlock { param ($m) msg * $m } -ArgumentList $Message
         Write-Host "Message sent!"
     } else {
         Write-Host "Computer not online"
     }
 
 }
`

