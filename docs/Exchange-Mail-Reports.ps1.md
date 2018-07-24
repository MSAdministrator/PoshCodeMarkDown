---
Author: andrei moraru
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5602
Published Date: 2014-11-17t22
Archived Date: 2014-12-03t06
---

# exchange mail reports - 

## Description

get csv report for messages received in mailbox during a month

## Comments



## Usage



## TODO



## function

`get-messagestomailboxforspecificmonth`

## Code

`#
 #
 function Get-MessagesToMailboxforSpecificMonth {
 
 <#
 .SYNOPSIS
     Determine the messages sent to mailbox since first day of month till the last day of month. Export the list with specific columns to .csv file 
  
 .DESCRIPTION
     Get-MessagesToITSforSpecificMonth is a function that returns the list with details of message sent to a mailbox during a month
  
 .PARAMETER Month
     The month to extract details of messages. The default is the previous month.
 
 .PARAMETER Mailbox
     The mailbox to extract details of messages. There is no default, it is mandatory parameter.
 
 .EXAMPLE
     Get-MessagesToITSforSpecificMonth -Mailbox am@contoso.com -Server nice-server.corp.contoso.com
  
 .EXAMPLE
      Get-MessagesToITSforSpecificMonth -Month October -Mailbox am@contoso.com
  
 .INPUTS
     Month, Mailbox
  
 .OUTPUTS
     File in the path where script is called
  
 .NOTES
     Author:  Andrei Moraru
     Website: http://develam.com
     Twitter: andrei.moraru.n@gmail.com
 #>
 
 param (
 [string]$Month,
 [ValidateNotNullorEmpty()][string]$Mailbox
 )
 
 Add-PSSnapin -Name Microsoft.Exchange.Management.Powershell.SnapIn
 
 $year = Get-Date -Format "yyyy"
 
 $arrMonths31days = "January","March","May","July","August","October","December"
 
 $arrMonths30days = "April","June","September","November"
 
 $outOfMonth = "Incorrect name of month, please specify English month name, i.e. January, February, March, etc."
 
 $noMailbox = "Mailbox $Mailbox doesn't exist in Exchange organization"
 
 
 if(-not ($Mailbox))
     {
     Write-Host -ForegroundColor Red "You must supply a value for -Mailbox"
     return
     } elseif(Get-Mailbox $Mailbox -ErrorAction SilentlyContinue)
         {}
     else {
     Write-Host -ForegroundColor Red $noMailbox
     return
     }
     
 
 if (-Not $Month) {$Month = (Get-Date).AddMonths(-1).ToString('MMMM')}
 
 
 
 if($arrMonths31days -contains $Month) 
     {$endMonth = 31}
 
     elseif($arrMonths30days -contains $Month) 
     {$endMonth = 30}
 
     elseif ($Month -eq "February")
     {$endMonth = 28}
 
     else {
     Write-Host -ForegroundColor Red $outOfMonth
     return
     }
 
 $fileName = "_"+$Month+"_"+$Mailbox+"_"+(Get-Date -Format "MMMdd_HHmmss")+".csv"
 
 $dbName = ((Get-Mailbox $Mailbox).Database).Name
 
 $arrServers = ((Get-MailboxDatabase -Identity $dbName).Servers).Name
 
 foreach($server in $arrServers){
     Get-MessageTrackingLog -Server $server -Start "1 $Month $year 00:00:00" -End "$endMonth $Month $year 23:59:59" -Recipients $Mailbox -ResultSize Unlimited | Select-Object Timestamp,RecipientCount,MessageSubject,Sender | Export-Csv ($server+$fileName) -Encoding ASCII -NoTypeInformation
     }
 }
`

