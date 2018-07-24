---
Author: rfoust@duke.edu
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3421
Published Date: 2013-05-20t18
Archived Date: 2016-04-06t09
---

# get exchange db stats - 

## Description

gathers lots of statistics about exchange 2010 databases, and sends an email with both a html table, and also css graphs representing disk usage.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 #
 #
 #
 #
 
 param([string]$server=$env:computername.tolower(), [string]$database="NotSpecified", [switch]$all, [switch]$noemail, [switch]$nologcheck, [switch]$nomountpoint)
 
 if (!(get-pssnapin Microsoft.Exchange.Management.PowerShell.E2010 -erroraction silentlycontinue))
 	{
 	Add-PSSnapin Microsoft.Exchange.Management.PowerShell.E2010
 	}
 
 function Util-Convert-FileSizeToString {	 
 	    param (
 	        [Parameter(Mandatory=$true, ValueFromPipeline=$true, Position=0)]
 	        [int64]$sizeInBytes
 	    )
 	 
 	    switch ($sizeInBytes)
 	    {
 	        {$sizeInBytes -ge 1TB} {"{0:n$sigDigits}" -f ($sizeInBytes/1TB) + " TB" ; break}
 	        {$sizeInBytes -ge 1GB} {"{0:n$sigDigits}" -f ($sizeInBytes/1GB) + " GB" ; break}
 	        {$sizeInBytes -ge 1MB} {"{0:n$sigDigits}" -f ($sizeInBytes/1MB) + " MB" ; break}
 	        {$sizeInBytes -ge 1KB} {"{0:n$sigDigits}" -f ($sizeInBytes/1KB) + " KB" ; break}
 	        Default { "{0:n$sigDigits}" -f $sizeInBytes + " Bytes" }
 	    }
 	}
 
 
 if ($database -eq "NotSpecified")
 	{
 	$dbs = get-mailboxdatabase -status | ? { $_.replicationtype -eq "Remote" } | sort servername,name
 	}
 else
 	{
 	$dbs = get-mailboxdatabase $database -status | sort servername,name
 	}
 
 
 
 foreach ($db in $dbs)
 	{
 	write-host "Processing $db."
 	
 	$raw = new-object psobject
 	$pretty = new-object psobject
 
 	write-host "Gathering count of mailboxes on $db."
 	$mailboxes = (get-mailbox -database $db.name -resultsize unlimited).count
 
 	$copystatus = (get-mailboxdatabasecopystatus $db.name | ? { $_.activecopy -eq $false }).status
 
 	if (-not $nologcheck)
 		{
 		write-host "Finding out how much disk space the log files are using for $db ..."
 		$drive = $db.logfolderpath.tostring().substring(0,1) + "$"
 		$substring = $db.logfolderpath.tostring().substring(0,2)
 		$uncpath = "\\$($db.server)\" + $drive + ($db.logfolderpath.tostring().replace($substring,"")) + "\*.log"
 		$logsize = (ls $uncpath | measure-object -property length -sum).sum
 		$logsizetotal += $logsize
 		$logsize = util-convert-filesizetostring $logsize
 		}
 	
 	Write-Host "Calculating average mailbox size ..."
 	$avg = get-mailboxstatistics -database $db | % { $_.totalitemsize.value.tobytes() }
 	$avg = ($avg | Measure-Object -Average).average
 	$avgTotal += $avg
 
 	if ($avg)
 		{
 		$avgPretty = util-convert-filesizetostring $avg
 		}
 		
 	Write-Host "Calculating deleted mailbox size ..."
 	$deletedMBsize = get-mailboxstatistics -database $db | ? { $_.DisconnectDate -ne $null } | % { $_.totalitemsize.value.tobytes() }
 	$deletedMBsize = ($deletedMBsize | Measure-Object -Sum).sum
 	$deletedMBsizeTotal += $deletedMBsize
 	
 	if ($deletedMBsize)
 		{
 		$deletedMBsizePretty = util-convert-filesizetostring $deletedMBsize
 		}
 	
 	Write-Host "Calculating dumpster size ..."
 	$dumpster = get-mailboxstatistics -database $db | % { $_.totaldeleteditemsize.value.tobytes() }
 	$dumpster = ($dumpster | Measure-Object -Sum).sum
 	$dumpsterTotal += $dumpster
 	
 	if ($dumpster)
 		{
 		$dumpsterPretty = util-convert-filesizetostring $dumpster
 		}
 	
 	$dbsize = $db.databasesize.tobytes()
 	$dbsizetotal += $dbsize
 	if ($dbsize)
 		{
 		$dbsizePretty = util-convert-filesizetostring $dbsize
 		}
 
 	if ($nomountpoint)
 		{
 		$freespace = (gwmi win32_volume -computer $db.server | where { $_.name -eq ($db.logfolderpath.drivename + "\") }).freespace
 		}
 	else
 		{
 		$freespace = (gwmi win32_volume -computer $db.server | where { $_.name -eq ($db.logfolderpath.tostring() + "\") }).freespace
 		}
 	$freespacetotal += $freespace
 	if ($freespace)
 		{
 		$freespacePretty = util-convert-filesizetostring $freespace
 		}
 
 	if ($nomountpoint)
 		{
 		$capacity = (gwmi win32_volume -computer $db.server | where { $_.name -eq ($db.logfolderpath.drivename + "\") }).capacity
 		}
 	else
 		{
 		$capacity = (gwmi win32_volume -computer $db.server | where { $_.name -eq ($db.logfolderpath.tostring() + "\") }).capacity
 		}
 	$capacitytotal += $capacity
 	if ($capacity)
 		{
 		$capacityPretty = util-convert-filesizetostring $capacity
 		}
 
 	$whitespace = $db.availablenewmailboxspace.tobytes()
 	$whitespacetotal += $whitespace
 	if ($whitespace)
 		{
 		$whitespacePretty = util-convert-filesizetostring $whitespace
 		}
 
 
 	$raw | add-member NoteProperty "ServerName" $db.servername
 	$raw | add-member NoteProperty "Database" $db.name
 	$raw | add-member NoteProperty "Mailboxes" $mailboxes
 	$raw | add-member NoteProperty "CopyStatus" $copystatus
 	$raw | add-member NoteProperty "DBSize" $dbsize
 	$raw | add-member NoteProperty "LogSize" $logsize
 	$raw | add-member NoteProperty "FreeSpace" $freespace
 	$raw | add-member NoteProperty "TotalSpace" $capacity
 	$raw | add-member NoteProperty "Whitespace" $whitespace
 	$raw | add-member NoteProperty "Deleted Mbox" $deletedMBsize
 	$raw | add-member NoteProperty "Dumpster" $dumpster
 	$raw | add-member NoteProperty "Avg Mbox" $avgPretty
 	$raw | add-member NoteProperty "Last Full Backup" $db.lastfullbackup
 	$raw | add-member NoteProperty "Last Incr Backup" $db.lastincrementalbackup
 
 	$data += $raw
 
 	$pretty | add-member NoteProperty "ServerName" $db.servername
 	$pretty | add-member NoteProperty "Database" $db.name
 	$pretty | add-member NoteProperty "Mailboxes" $mailboxes
 	$pretty | add-member NoteProperty "CopyStatus" $copystatus
 	$pretty | add-member NoteProperty "DBSize" $dbsizePretty
 	$pretty | add-member NoteProperty "LogSize" $logsize
 	$pretty | add-member NoteProperty "FreeSpace" $freespacePretty
 	$pretty | add-member NoteProperty "TotalSpace" $capacityPretty
 	$pretty | add-member NoteProperty "Whitespace" $whitespacePretty
 	$pretty | add-member NoteProperty "Deleted Mbox" $deletedMBsizePretty
 	$pretty | add-member NoteProperty "Dumpster" $dumpsterPretty
 	$pretty | add-member NoteProperty "Avg Mbox" $avgPretty
 	$pretty | add-member NoteProperty "Last Full Backup" $db.lastfullbackup
 	$pretty | add-member NoteProperty "Last Incr Backup" $db.lastincrementalbackup
 
 	$prettydata += $pretty
 	}
 
 $thingy = new-object psobject
 write-host; write-host "Calculating totals ..."
 
 $mailboxes = ($data | measure-object mailboxes -sum).sum
 $deletedMBsizetotal = ($deletedMBsizetotal | Measure-Object -Sum).sum
 
 $thingy | add-member NoteProperty "ServerName" "Total"
 $thingy | add-member NoteProperty "Database" $data.count
 $thingy | add-member NoteProperty "DBSize" (util-convert-filesizetostring $dbsizetotal)
 $thingy | add-member NoteProperty "Mailboxes" $mailboxes
 if (-not $nologcheck)
 	{
 	$thingy | add-member NoteProperty "LogSize" (util-convert-filesizetostring $logsizetotal)
 	}
 $thingy | add-member NoteProperty "FreeSpace" (util-convert-filesizetostring $freespacetotal)
 $thingy | add-member NoteProperty "TotalSpace" (util-convert-filesizetostring $capacitytotal)
 $thingy | add-member NoteProperty "Whitespace" (util-convert-filesizetostring $whitespacetotal)
 $thingy | Add-Member NoteProperty "Deleted Mbox" (util-convert-filesizetostring $deletedMBsizeTotal)
 $thingy | Add-Member NoteProperty "Dumpster" (util-convert-filesizetostring $dumpsterTotal)
 #$thingy | Add-Member NoteProperty "Avg Mbox" (util-convert-filesizetostring $avgTotal)
 
 $prettyData += $thingy
 
 $thingy = new-object psobject
 
 $mailboxes = ($data | measure-object mailboxes -sum).sum
 $deletedMBsizetotal = ($deletedMBsizetotal | Measure-Object -Sum).sum
 
 $thingy | add-member NoteProperty "ServerName" "Total"
 $thingy | add-member NoteProperty "Database" $data.count
 $thingy | add-member NoteProperty "DBSize" $dbsizetotal
 $thingy | add-member NoteProperty "Mailboxes" $mailboxes
 if (-not $nologcheck)
 	{
 	$thingy | add-member NoteProperty "LogSize" $logsizetotal
 	}
 $thingy | add-member NoteProperty "FreeSpace" $freespacetotal
 $thingy | add-member NoteProperty "TotalSpace" $capacitytotal
 $thingy | add-member NoteProperty "Whitespace" $whitespacetotal
 $thingy | Add-Member NoteProperty "Deleted Mbox" $deletedMBsizeTotal
 $thingy | Add-Member NoteProperty "Dumpster" $dumpsterTotal
 #$thingy | Add-Member NoteProperty "Avg Mbox" $avgTotal
 
 $data += $thingy
 
 $prettyData
 
 $style = "<style type=`"text/css`">
 html * { margin: 0; padding: 0; border: 0; }
 body { text-align: left; font: 10pt Arial, sans-serif; }
 TH { border-width: 1px; padding: 0px; border-style: solid; border-color: black; background-color: thistle }
 h2 { font: 20pt Arial, sans-serif; }
 a:link, a:visited, a:hover { color: rgb(32,108,223); font-weight: bold; text-decoration: none; }
 
 
 
 /*	BULLET GRAPH */
 div.box-wrap { position: relative; width: 200px; height: 21px; top: 0; left: 0; margin: 0; padding: 0; }
 
 /*	CHANGE THE WIDTH AND BACKGROUND COLORS AS NEEDED */
 
 /* RED LINE */
 
 /* ONE SEGMENT ONLY */
 
 /* TWO SEGMENTS */
 
 div.mylabel { 
 	position: relative; 
 	height: 20px; 
 	width: 145px; 
 	left: -155px; 
 	top: 2px; 
 	z-index: 7; 
 	font-size: 0;
 	font: 10pt Arial, sans-serif; 
 	text-align: right; 
 }
 
 div.scale-tb1 {
 	padding: 0; 
 	margin: 0;
 	font-size: 0;
 	width: 200px;
 	border: 0;	
 	position: relative; 
 	top: 10px; 
 	left: 0px; 
 }
 
 div.scale-tb2 {
 	padding: 0; 
 	margin: 0;
 	font-size: 0;
 	width: 200px;
 	border: 0;	
 	position: relative; 
 	top: 0px; 
 	left: 0px; 
 }
 
 /*	SCALE MARKS */
 
 
 
 /*	SCALE TEXT */
 
 </style>"
 
 $fragments += "<H2>Exchange 2010 Statistics</H2>"
 $fragments += $prettyData | ConvertTo-Html -fragment
 
 $html = @()
 
 $html += "<div id=`"container`">"
 $html += "Database Size Graph"
 
 foreach ($db in $data)
 	{
 	if ($db.servername -ne "Total")
 		{		
 		$html += "<div class=`"box-wrap`">
 		<div class=`"box1`"></div>
 		<div class=`"box2`"></div>
 		<div class=`"box3`"></div>
 		<div class=`"box4`"></div>
 		<div class=`"target`" style=`"left: $([math]::round((($db.totalspace - $db.freespace) / $db.totalspace) * 100))%`"></div>
 		<div class=`"actualWhitespace`" style=`"width: $([math]::round((($db.dbsize) / $db.totalspace)*100))%`"></div>
 		<div class=`"actualDeleted`" style=`"width: $([math]::round((($db.dbsize - $db.whitespace) / $db.totalspace)*100))%`"></div>
 		<div class=`"actualDumpster`" style=`"width: $([math]::round((($db.dbsize - $db.whitespace - $db.'deleted mbox') / $db.totalspace)*100))%`"></div>
 		<div class=`"actualData`" style=`"width: $([math]::round((($db.dbsize - $db.dumpster - $db.'deleted mbox' - $db.whitespace) / $db.totalspace)*100))%`"></div>
 		<div class=`"mylabel`">$($db.database)</div>
 
 		<div class=`"cap31`">0%</div>
 		<div class=`"cap32`">20%</div>
 		<div class=`"cap33`">40%</div>
 		<div class=`"cap34`">60%</div>
 		<div class=`"cap35`">80%</div>
 		<div class=`"cap36`">100%</div>	
 
 		<div class=`"scale-tb2`">
 		<div class=`"sc31`"></div>
 		<div class=`"sc32`"></div>
 		<div class=`"sc33`"></div>
 		<div class=`"sc34`"></div>
 		<div class=`"sc35`"></div>
 		<div class=`"sc36`"></div>
 		</div>	
 
 		</div>
 
 		<p style=`"height: 30px;`"></p>"
 		}
 		{
 		$html += "<div class=`"box-wrap`">
 		<div class=`"box1`"></div>
 		<div class=`"box2`"></div>
 		<div class=`"box3`"></div>
 		<div class=`"box4`"></div>
 		<div class=`"target`" style=`"left: $([math]::round((($db.totalspace - $db.freespace) / $db.totalspace) * 100))%`"></div>
 		<div class=`"actualWhitespace`" style=`"width: $([math]::round((($db.dbsize) / $db.totalspace)*100))%`"></div>
 		<div class=`"actualDeleted`" style=`"width: $([math]::round((($db.dbsize - $db.whitespace) / $db.totalspace)*100))%`"></div>
 		<div class=`"actualDumpster`" style=`"width: $([math]::round((($db.dbsize - $db.whitespace - $db.'deleted mbox') / $db.totalspace)*100))%`"></div>
 		<div class=`"actualData`" style=`"width: $([math]::round((($db.dbsize - $db.dumpster - $db.'deleted mbox' - $db.whitespace) / $db.totalspace)*100))%`"></div>
 		<div class=`"mylabel`">Total</div>
 
 		<div class=`"cap31`">0%</div>
 		<div class=`"cap32`">20%</div>
 		<div class=`"cap33`">40%</div>
 		<div class=`"cap34`">60%</div>
 		<div class=`"cap35`">80%</div>
 		<div class=`"cap36`">100%</div>	
 
 		<div class=`"scale-tb2`">
 		<div class=`"sc31`"></div>
 		<div class=`"sc32`"></div>
 		<div class=`"sc33`"></div>
 		<div class=`"sc34`"></div>
 		<div class=`"sc35`"></div>
 		<div class=`"sc36`"></div>
 		</div>	
 
 		</div>
 
 		<p style=`"height: 30px;`"></p>"
 		}
 	}
 
 $html += "<div class=`"box-wrap`">
 		<div class=`"box1`"></div>
 		<div class=`"box2`"></div>
 		<div class=`"box3`"></div>
 		<div class=`"box4`"></div>
 		<div class=`"target`" style=`"left: 90%`"></div>
 		<div class=`"actualWhitespace`" style=`"width: 100%`"></div>
 		<div class=`"actualDeleted`" style=`"width: 75%`"></div>
 		<div class=`"actualDumpster`" style=`"width: 50%`"></div>
 		<div class=`"actualData`" style=`"width: 25%`"></div>
 		<div class=`"mylabel`">Key:</div>
 
 		</div>	
 		MBs - Dumpster - Del MB - Whitespace<br>
 		Red Line: Used Disk Space
 		</div>
 
 		<p style=`"height: 30px;`"></p>"
 
 $html += "</div><!-- container -->"
 $fragments += $html	
 $emailbody = convertto-html -head $style -Body $fragments
 $date = get-date -uformat "%Y-%m-%d"
 
 if (-not $noemail)
 	{
 	$smtpServer = "your.smtpserver.com"
 	$msg = new-object Net.Mail.MailMessage
 	$smtp = new-object Net.Mail.SmtpClient($smtpServer)
 	$msg.From = "somebody@somewhere.com"
 	$msg.To.Add("you@somewhere.com")
 	$msg.Subject = "Exchange 2010 Daily Statistics"
 	$msg.Body = $emailbody
 	$msg.IsBodyHtml = $true
 
 	$smtp.Send($msg)
 	}
`

