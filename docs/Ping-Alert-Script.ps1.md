---
Author: themoblin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6221
Published Date: 2016-02-16t10
Archived Date: 2016-09-09t00
---

# ping alert script - 

## Description

will alert on failed pings and again when recovered.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 $to = "user@mydomain.com"
 
 $from = "unreachable@mydomain.com"
 
 $smtpserver = "my_exchange_server"
 
  
 
 
 $Computers = ("comp1" , "comp2" , "comp3" , "comp4")
 
 $zero = 0
 
 Foreach ($Computer in $Computers)
 
     {
 
     if
 
         (
         Test-Path $("C:\Reports\" + $Computer + ".txt")
 
         )
 
         {
 
         }
 
         else
 
         {
 
         $zero > $("C:\Reports\" + $Computer + ".txt")
 
         }
     $FailedPings = Get-Content $("C:\Reports\" + $Computer + ".txt")
     $INT_FailedPings  = [INT]$FailedPings
     $PingTest = Test-Connection -ComputerName $Computer -count 1
 
     if
 
         (
         $PingTest.StatusCode -ne "0"
 
         )
 
         {
 
     if
 
         (
         $INT_FailedPings  -le 3
 
         )
 
         {
         $INT_FailedPings++
         $INT_FailedPings  > $("C:\Reports\"  + $Computer  + ".txt")
         Send-MailMessage -to $to -subject "Warning, Host $Computer is down. You will only receive 4 of these messages!" -from $from  -body "Ping to $Computer failed, you will only receive 3 of these messages!" -smtpserver $smtpserver
 
         }
 
         }
 
    elseif
 
         (
         $INT_FailedPings  -ne 0
 
         )
 
         {
 
         $zero > $("C:\Reports\" + $Computer + ".txt")
 
         Send-MailMessage -to $to -subject "Host $Computer is back up" -from $from  -body "Panic over"  -smtpserver $smtpserver
 
         }
 
     else
         {
 
         }
 
     }
`

