---
Author: jan egil ring
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2427
Published Date: 2013-12-30t03
Archived Date: 2013-06-23t06
---

# new-dfsrhealthreport - 

## Description

a windows powershell-script for generating an html-report for dfs-r sysvol and sending it via e-mail.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 
  NAME: New-DFSRHealthReport.ps1
 
  AUTHOR: Jan Egil Ring
  EMAIL: jan.egil.ring@powershell.no
 
  COMMENT: Script to generate a new DFS-R HTML Health Report for the Domain System Volume (SYSVOL) replication group using the native dfsradmin.exe tool.
           More information: 
 
  You have a royalty-free right to use, modify, reproduce, and
  distribute this script file in any way you find useful, provided that
  you agree that the creator, owner above has no warranty, obligations,
  or liability for such use.
 
  VERSION HISTORY:
  1.0 30.12.2010 - Initial release
 
 #>
 
 
 $ReportFilePath = "C:\Reports\DFS-R\Health-report-"+(Get-Date -UFormat %Y-%m-%d)
 $ReferralDC = "domain\srv-dc-01"
 $SmtpServer = "srv-smtp-01.domain.local"
 $EmailFrom = "dfs-r@domain.local"
 $EmailTo = "user@domain.local"
 
 DfsrAdmin.exe Health New /RgName:`"Domain System Volume`" /RefMemName:$ReferralDC /RepName:$ReportFilePath /FsCount:true
 
 Send-MailMessage -From $EmailFrom -To $EmailTo -SmtpServer $SmtpServer -Subject "DFS-R Health Report 2010-12-30" -Body "The DFS-R Health Report is attched to this e-mail" -Attachments ($ReportFilePath+".html")
`

