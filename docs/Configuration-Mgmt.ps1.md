---
Author: lubinski
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4521
Published Date: 2013-10-15t17
Archived Date: 2013-10-21t07
---

# configuration mgmt - 

## Description

a configuration management script for when changes occur to a vmware view environment. change the color codes to suit your needs.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 $Data = @()
 
 Function writeHtmlHeader
 {
 param($title, $fileName)
 $date = ( Get-Date ).ToString('dddd dd/MM/yyyy')
 $head = @"
 <html>
 <head>
 <meta http-equiv='Content-Type' content='text/html; charset=iso-8859-1'>
 <title>$title</title>
 <STYLE TYPE="text/css">
 <!--
 td {
 font-family: Tahoma;
 font-size: 11px;
 padding-top: 0px;
 padding-right: 0px;
 padding-bottom: 0px;
 padding-left: 0px;
 }
 body {
 margin-left: 5px;
 margin-top: 5px;
 margin-right: 0px;
 margin-bottom: 10px;
 table {
 }
 -->
 </style>
 </head>
 <body>
 <table width='800'>
 <td colspan='1' height='25' align='center'>
 </td>
 </tr>
 </table>
 "@
 $head | Out-File $fileName
 }
 
 Function writeTableHeader
 {
 param($fileName)
 $tableHeader = @"
 <table width='800'><tbody>
 <td width=10% align='center'>Time</td>
 <td width=20% align='center'>Event Type</td>
 <td width=10% align='center'>Severity </td>
 <td width=20% align='center'>Username</td>
 <td width=40% align='center'>Text</td>
 </tr>
 "@
 $tableHeader | Out-File $fileName -append
 }
 
 Function writeData
 {
                 param([string[]]$infoColumns, [string[]]$rowData, $fileName)
                 
                 $tableEntry = "<tr>"
                 $infoColumns | % {
                 }
                 
                 $rowData | % {                              
                 $tableEntry += "</tr>"
                 $tableEntry | Out-File $fileName -append
 }
 
 Function writeHtmlFooter
 {
 param($fileName)
 @"
 </body>
 </html>
 "@ | Out-File $FileName -append
 }
 
 add-PSSnapin vmware.view.broker
 $username = "username"
 $password = "password"
 $domain = "domain"
 $VCS = "view connection server address"
 
 $CurrentDate = Get-Date
 $DateSelection = ($CurrentDate).addhours(-1)
 $exportfile = 'c:\pathtoexportfile.html'
 $AllEvents = Get-EventReport -ViewName config_changes -StartDate $DateSelection | select time,eventtype,severity,userdisplayname,moduleandeventtext
 
 If ($AllEvents -ne $null) {
 writeHtmlHeader "VDI Configuration Change Report" $ExportFile
 writeTableHeader $ExportFile
 $Allevents | % { writedata -fileName $exportfile -rowData $_.time,$_.eventtype,$_.severity,$_.userdisplayname,$_.moduleandeventtext }
 writeHtmlFooter $ExportFile
 
 $smtpServer = "smtp.yourserver.com"
 $msg = new-object Net.Mail.MailMessage
 $smtp = new-object Net.Mail.SmtpClient($smtpServer)
 $msg.From = "VDIConfigChecker@something.com"
 $msg.To.Add("youremailhere@company.com")
 $msg.IsBodyHtml = $true
 $msg.Subject = "VDI Configuration Change Notification"
 $msg.Body = (gc $exportfile)
 $smtp.Send($msg)
 }
 Remove-Item $exportfile
`

