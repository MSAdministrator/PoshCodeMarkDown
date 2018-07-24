---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4456
Published Date: 2014-09-11t11
Archived Date: 2014-08-18t21
---

# search sql error log - search-sqlerrorlog.ps1

## Description

#############################################################################################

## Comments



## Usage



## TODO



## function

`search-sqlerrorlog`

## Code

`#
 #
   #############################################################################################
 #
 #
 #
 
    Output to file
         Search-SQLErrorLog backup fade2black|Out-File c:\temp\error.txt
 
    Email
         $ErrorLog = Search-SQLErrorLog FADE2BLACK
         Send-MailMessage -SmtpServer Mail.Server -to DBA@MyWork.com -Subject SQL Error Log Search for $Search -Body $ErrorLog 
 
    Email as attachment
         $ErrorLog = Search-SQLErrorLog FADE2BLACK|Out-File c:\temp\log.txt
         $Body = @"
                   Dear DBA, 
 
                   Please find attached the Error Log for $Search. 
 
                   You're Welcome. 
 
                   The Beard
                 "@
          Send-MailMessage -SmtpServer Mail.Server -to DBA@MyWork.com -Subject "SQL Error Log" `
          -Body $Body -Attachments $ErrorLog
 #>
 
 Function Search-SQLErrorLog ([string] $SearchTerm , [string] $SQLServer)
  {
 
  $Search = '*' + $SearchTerm + '*'
 
  
 
 $server = new-object "Microsoft.SqlServer.Management.Smo.Server" $SQLServer
 
  
 $server.ReadErrorLog(6)| Where-Object {$_.Text -like $Search} | Select LogDate, ProcessInfo, Text| Format-Table -Wrap -AutoSize
 $server.ReadErrorLog(5)| Where-Object {$_.Text -like $Search} | Select LogDate, ProcessInfo, Text| Format-Table -Wrap -AutoSize
 $server.ReadErrorLog(4)| Where-Object {$_.Text -like $Search} | Select LogDate, ProcessInfo, Text| Format-Table -Wrap -AutoSize
 $server.ReadErrorLog(3)| Where-Object {$_.Text -like $Search} | Select LogDate, ProcessInfo, Text| Format-Table -Wrap -AutoSize
 $server.ReadErrorLog(2)| Where-Object {$_.Text -like $Search} | Select LogDate, ProcessInfo, Text| Format-Table -Wrap -AutoSize 
 $server.ReadErrorLog(1)| Where-Object {$_.Text -like $Search} | Select LogDate, ProcessInfo, Text| Format-Table -Wrap -AutoSize 
 $server.ReadErrorLog(0)| Where-Object {$_.Text -like $Search} | Select LogDate, ProcessInfo, Text| Format-Table -Wrap -AutoSize 
 
 
 }
`

