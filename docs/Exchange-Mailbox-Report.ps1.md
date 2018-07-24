---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 4.01
Encoding: utf-8
License: cc0
PoshCode ID: 2464
Published Date: 
Archived Date: 2011-01-22t09
---

# exchange mailbox report - 

## Description

this file was uploaded by a powergui script editor add-on.

## Comments



## Usage



## TODO



## function

`send-smtpmail`

## Code

`#
 #
 function Send-SMTPmail($to, $from, $subject, $smtpserver, $body) {
 	$mailer = new-object Net.Mail.SMTPclient($smtpserver)
 	$msg = new-object Net.Mail.MailMessage($from,$to,$subject,$body)
 	$msg.IsBodyHTML = $true
 	$mailer.send($msg)
 }
 
 Function Get-CustomHTML ($Header){
 $Report = @"
 <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">
 <html><head><title>$($Header)</title>
 		<META http-equiv=Content-Type content='text/html; charset=windows-1252'>
 
 		<style type="text/css">
 
 		TABLE 		{
 						TABLE-LAYOUT: fixed; 
 						FONT-SIZE: 100%; 
 						align: center;
 					}
 		*
 					{
 						margin:0
 					}
 
 		.dspcont 	{
 	
 						PADDING-LEFT: 0px;
 						FONT-SIZE: 8pt;
 						MARGIN-BOTTOM: -1px;
 						PADDING-BOTTOM: 5px;
 						MARGIN-LEFT: 0px;
 						WIDTH: 95%;
 						MARGIN-RIGHT: 0px;
 						PADDING-TOP: 4px;
 						FONT-FAMILY: Tahoma;
 						POSITION: relative;
 					}
 					
 		.filler 	{
 						BORDER-RIGHT: medium none; 
 						BORDER-TOP: medium none; 
 						DISPLAY: block; 
 						BACKGROUND: none transparent scroll repeat 0% 0%; 
 						MARGIN-BOTTOM: -1px; 
 						FONT: 100%/8px Tahoma; 
 						MARGIN-LEFT: 43px; 
 						BORDER-LEFT: medium none; 
 						MARGIN-RIGHT: 0px; 
 						PADDING-TOP: 4px; 
 						BORDER-BOTTOM: medium none; 
 						POSITION: relative
 					}
 
 		.pageholder	{
 						margin: 0px auto;
 					}
 					
 		.dsp
 					{
 						PADDING-RIGHT: 0px;
 						DISPLAY: block;
 						PADDING-LEFT: 0px;
 						FONT-WEIGHT: bold;
 						FONT-SIZE: 8pt;
 						MARGIN-BOTTOM: -1px;
 						MARGIN-LEFT: 0px;
 						MARGIN-RIGHT: 0px;
 						PADDING-TOP: 4px;
 						FONT-FAMILY: Tahoma;
 						POSITION: relative;
 						HEIGHT: 2.25em;
 						WIDTH: 95%;
 						TEXT-INDENT: 10px;
 					}
 
 		.dsphead0	{
 						BACKGROUND-COLOR: #$($Colour1);
 					}
 					
 		.dsphead1	{
 						
 						BACKGROUND-COLOR: #$($Colour2);
 					}
 					
 	.dspcomments 	{
 						FONT-STYLE: ITALIC;
 						FONT-WEIGHT: normal;
 						FONT-SIZE: 8pt;
 					}
 
 	td 				{
 						VERTICAL-ALIGN: TOP; 
 						FONT-FAMILY: Tahoma;
 					}
 					
 	th 				{
 						VERTICAL-ALIGN: TOP; 
 						COLOR: #$($Colour1); 
 						TEXT-ALIGN: left;
 					}
 					
 	BODY 			{
 						margin-left: 4pt;
 						margin-right: 4pt;
 						margin-top: 6pt;
 					} 
 	.MainTitle		{
 						font-family:Arial, Helvetica, sans-serif;
 						font-size:20px;
 						font-weight:bolder;
 					}
 	.SubTitle		{
 						font-family:Arial, Helvetica, sans-serif;
 						font-size:14px;
 						font-weight:bold;
 					}
 	.Created		{
 						font-family:Arial, Helvetica, sans-serif;
 						font-size:10px;
 						font-weight:normal;
 						margin-top: 20px;
 						margin-bottom:5px;
 					}
 	.links			{	font:Arial, Helvetica, sans-serif;
 						font-size:10px;
 						FONT-STYLE: ITALIC;
 					}
 					
 		</style>
 	</head>
 	<body>
 <div class="MainTitle">$($Header)</div>
         <hr size="8" color="#$($Colour1)">
         <div class="SubTitle">Generated on $($ENV:Computername)</div>
 	    <br/>
 		<div class="Created">Report created on $(Get-Date)</div>
 "@
 Return $Report
 }
 
 Function Get-CustomHeader0 ($Title){
 $Report = @"
 		<div class="pageholder">		
 
 		<h1 class="dsp dsphead0">$($Title)</h1>
 	
     	<div class="filler"></div>
 "@
 Return $Report
 }
 
 Function Get-CustomHeader ($Title, $cmnt){
 $Report = @"
 	    <h2 class="dsp dsphead1">$($Title)</h2>
 "@
 If ($Comments) {
 	$Report += @"
 			<div class="dsp dspcomments">$($cmnt)</div>
 "@
 }
 $Report += @"
         <div class="dspcont">
 "@
 Return $Report
 }
 
 Function Get-CustomHeaderClose{
 
 	$Report = @"
 		</DIV>
 		<div class="filler"></div>
 "@
 Return $Report
 }
 
 Function Get-CustomHeader0Close{
 	$Report = @"
 </DIV>
 "@
 Return $Report
 }
 
 Function Get-CustomHTMLClose{
 	$Report = @"
 </div>
 
 </body>
 </html>
 "@
 Return $Report
 }
 
 Function Get-HTMLTable {
 	param([array]$Content)
 	$HTMLTable = $Content | ConvertTo-Html
 	$HTMLTable = $HTMLTable -replace '<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">', ""
 	$HTMLTable = $HTMLTable -replace '<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"  "http://www.w3.org/TR/html4/strict.dtd">', ""
 	$HTMLTable = $HTMLTable -replace '<html xmlns="http://www.w3.org/1999/xhtml">', ""
 	$HTMLTable = $HTMLTable -replace '<html>', ""
 	$HTMLTable = $HTMLTable -replace '<head>', ""
 	$HTMLTable = $HTMLTable -replace '<title>HTML TABLE</title>', ""
 	$HTMLTable = $HTMLTable -replace '</head><body>', ""
 	$HTMLTable = $HTMLTable -replace '</body></html>', ""
 	$HTMLTable = $HTMLTable -replace '&lt;', "<"
 	$HTMLTable = $HTMLTable -replace '&gt;', ">"
 	Return $HTMLTable
 }
 
 Function Get-HTMLDetail ($Heading, $Detail){
 $Report = @"
 <TABLE>
 	<tr>
 	<th width='50%'><b>$Heading</b></font></th>
 	<td width='50%'>$($Detail)</td>
 	</tr>
 </TABLE>
 "@
 Return $Report
 }
 $Date = Get-Date -Format g
 $Comments = $CommentsSet
 $Comments = $false
 
 $exchserver = "nau-xs2.naucom.com"
 $allmailboxes = ((Get-MailboxStatistics -Server "$exchserver"))
 $activemail = Get-MailboxStatistics -Server "$exchserver" | where-object {$_.LastLogonTime -gt ((Get-Date).adddays(-30))} | Select-Object DisplayName,DatabaseName,ItemCount,LastLogonTime,TotalDeletedItemSize,TotalItemSize | Sort-Object TotalItemSize -Descending:$true
 $inactivemail = Get-MailboxStatistics -Server "$exchserver" | where-object {$_.LastLogonTime -lt ((Get-Date).adddays(-30))} | Select-Object DisplayName,DatabaseName,ItemCount,LastLogonTime,TotalDeletedItemSize,TotalItemSize | Sort-Object LastLogonTime -Descending:$true
 $activesize = $activemail | Measure-Object -Property TotalItemSize -Sum
 $activedel = $activemail | Measure-Object -Property TotalDeletedItemSize -Sum
 $inactivesize = $inactivemail | Measure-Object -Property TotalItemSize -Sum
 $inactivedel = $inactivemail | Measure-Object -Property TotalDeletedItemSize -Sum
 
 $MyReport = Get-CustomHTML "Exchange Mailbox Report"
 $MyReport += Get-CustomHeader0 "Exchange Mailbox Statistics"
 $MyReport += Get-CustomHeader "General Details" ""
 $MyReport += Get-HTMLDetail "Total Number of Mailboxes: " ($allmailboxes.Count)
 $MyReport += Get-HTMLDetail "Number of Active Mailboxes: " ($activemail.Count)
 $MyReport += Get-HTMLDetail "Number of Inactive Mailboxes: " ($inactivemail.Count)
 $MyReport += Get-HTMLDetail "Total Size of Active Mailboxes(GB): " ([math]::truncate($activesize.Sum / 1GB))
 $MyReport += Get-HTMLDetail "Deleted Item Size of Active Mailboxes(GB): " ([math]::truncate($activedel.Sum / 1GB))
 $MyReport += Get-HTMLDetail "Total Size of Inactive Mailboxes(GB): " ([math]::truncate($inactivesize.Sum / 1GB))
 $MyReport += Get-HTMLDetail "Deleted Item size of Inactive Mailboxes(GB): " ([math]::truncate($inactivedel.Sum / 1GB))
 $MyReport += Get-HTMLDetail "Average Mailbox Size(MB): " ([math]::truncate((($allmailboxes | Measure-Object -Property TotalItemSize -Average).Average) / 1MB))
 $MyReport += Get-CustomHeaderClose
 
 $Comments = $true
 $MyReport += Get-CustomHeader "Active Mailboxes" "These are mailboxes who have had activity within the last 30 days."
 	$HTMLTable = $activemail | ConvertTo-Html @{label=â��Userâ��;expression={$_.DisplayName}},@{label=â��Total Size (MB)â��;expression={$_.TotalItemSize.Value.ToMB()}},@{label=â��Deleted Size (MB)â��;expression={$_.TotalDeletedItemSize.Value.ToMB()}},@{label=â��Itemsâ��;expression={$_.ItemCount}},@{label=â��Databaseâ��;expression={$_.DataBaseName}},@{label="LastLogonâ��;expression={$_.LastLogonTime}}
 	$HTMLTable = $HTMLTable -replace '<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">', ""
 	$HTMLTable = $HTMLTable -replace '<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"  "http://www.w3.org/TR/html4/strict.dtd">', ""
 	$HTMLTable = $HTMLTable -replace '<html xmlns="http://www.w3.org/1999/xhtml">', ""
 	$HTMLTable = $HTMLTable -replace '<html>', ""
 	$HTMLTable = $HTMLTable -replace '<head>', ""
 	$HTMLTable = $HTMLTable -replace '<title>HTML TABLE</title>', ""
 	$HTMLTable = $HTMLTable -replace '</head><body>', ""
 	$HTMLTable = $HTMLTable -replace '</body></html>', ""
 	$HTMLTable = $HTMLTable -replace '&lt;', "<"
 	$HTMLTable = $HTMLTable -replace '&gt;', ">"
 $MyReport += $HTMLTable
 $MyReport += Get-CustomHeaderClose
 
 $Comments = $true
 $MyReport += Get-CustomHeader "Inactive Mailboxes" "These are mailboxes who have not logged in within the last 30 days."
 	$HTMLTable = $inactivemail | ConvertTo-Html @{label=â��Userâ��;expression={$_.DisplayName}},@{label=â��Total Size (MB)â��;expression={$_.TotalItemSize.Value.ToMB()}},@{label=â��Deleted Size (MB)â��;expression={$_.TotalDeletedItemSize.Value.ToMB()}},@{label=â��Itemsâ��;expression={$_.ItemCount}},@{label=â��Databaseâ��;expression={$_.DataBaseName}},@{label="LastLogonâ��;expression={$_.LastLogonTime}}
 	$HTMLTable = $HTMLTable -replace '<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">', ""
 	$HTMLTable = $HTMLTable -replace '<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"  "http://www.w3.org/TR/html4/strict.dtd">', ""
 	$HTMLTable = $HTMLTable -replace '<html xmlns="http://www.w3.org/1999/xhtml">', ""
 	$HTMLTable = $HTMLTable -replace '<html>', ""
 	$HTMLTable = $HTMLTable -replace '<head>', ""
 	$HTMLTable = $HTMLTable -replace '<title>HTML TABLE</title>', ""
 	$HTMLTable = $HTMLTable -replace '</head><body>', ""
 	$HTMLTable = $HTMLTable -replace '</body></html>', ""
 	$HTMLTable = $HTMLTable -replace '&lt;', "<"
 	$HTMLTable = $HTMLTable -replace '&gt;', ">"
 $MyReport += $HTMLTable
 $MyReport += Get-CustomHeaderClose
 
 $MyReport += Get-CustomHeader0Close
 $MyReport += Get-CustomHTMLClose
 
 $MyReport | Out-File H:\exreport.html
`

