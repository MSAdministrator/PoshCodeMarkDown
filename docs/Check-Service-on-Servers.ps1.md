---
Author: matt schmitt
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3803
Published Date: 2013-11-29t12
Archived Date: 2013-11-11t19
---

# check service on servers - 

## Description

if you download or use, please tweet it or email me.  thanks!

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
   Author:   Matt Schmitt
   Date:     11/29/12 
   Version:  1.0 
   From:     USA 
   Email:    ithink2020@gmail.com 
   Website:  http://about.me/schmittmatt
   Twitter:  @MatthewASchmitt
   
   Description
   A script for checking the status of a service on a group of servers, from a list in a file.  
 #>
 
 
 $serverList = Import-Csv 'c:\serverList.csv'
 
 "Server" +"`t" + "Status" | Out-File c:\ServerService.csv
 
 
 foreach ($element in $serverList) 
 {
     
     $sStatus = get-service -Name "CPSVS" | Select-Object -expand Status
 
     $server = $element | Select-Object -expand Server
 
     $server + "`t" + $sStatus | Out-File -append c:\ServerServiceStatus.csv
 
 } 
 
 
 Send-MailMessage -From donotreply@test.com -To recipient@domain.com -subject "Spooler Service Report" -Body "Attached is Server Service report." -Attachments "c:\ServerServiceStatus.csv" -SmtpServer "xxx.xxx.xxx.xxx"
`

