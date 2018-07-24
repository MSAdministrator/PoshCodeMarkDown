---
Author: samuel mulhearn
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3510
Published Date: 2012-07-11t06
Archived Date: 2012-07-17t20
---

# vm performance report - - 

## Description

the script requires an input file, supplied as an argument to the script. the first line of this file contains an email address, subsequent lines contain vm names, one per line. the script connects to vcenter, queries for given performance stats (get-stat) and then generates a html report, which is then emailed to the recipient. the script can be scheduled via scheduled tasks (when supplied with a credential file for vcenter created with http

## Comments



## Usage



## TODO



## script

`out-logfile`

## Code

`#
 #
 .SYNOPSIS 
     The script creates an HTML report for given vSphere VM's, that contains VM performance data over a given period. The script then emails the report to a given address.
 .DESCRIPTION 
 	The script requires an input file, supplied as an argument to the script. The first line of this file contains an email address, subsequent lines contain VM names, one per line. The script connects to vCenter, queries for given performance stats (get-stat) and then generates a HTML report, which is then emailed to the recipient. The script can be scheduled via scheduled tasks (when supplied with a credential file for vCenter created with http://poshcode.org/3491) or run interactively.
 .NOTES 
     File Name  : Get-VMPeformanceReport.ps1
     Author     : Samuel Mulhearn
     Version History: 
 	Version 1.0  
 		28 Jun 2012.
 		Release
 	Version 1.1
 		11 July 2012
 		Andy Helsby - Modified for US formatting of date and time
 .LINK 
     http://poshcode.org/3494
 .EXAMPLE 
     Call the script with a single argument which should be the path to the input file .\Get-VMPeformanceReport.ps1 <path\datafile.txt>
 #> 
 
 $CompanyLogo = "http://placehold.it/150x50"
 
 function Out-LogFile {
 	[CmdletBinding(DefaultParameterSetName='Message')]
 	param(
 		[Parameter(ParameterSetName='Message',
 		Position=0,
 		ValueFromPipeline=$true)]				
 		[object[]]$Message,
 		[Parameter(ParameterSetName='Message')]
 		[string]$LogFile = $global:DefaultLogPath,
 		[Parameter(ParameterSetName='Message')]
 		[int]$BlankLine = 0,		
 		[switch]$WriteHost = $global:WriteHostPreference,
 		[string]$Severity = "I",
 		[Parameter(ParameterSetName='Message')]
 		[switch]$DontFormat,		
 		[Parameter(ParameterSetName='Message')]
 		[string]$DateFormat = "dd-MM-yyyy HH:mm:ss",		
 		#[Parameter(ParameterSetName='Title',Position=0,Mandatory=$true)]
 		[string]$Title,		
 		[System.ConsoleColor]$ForegroundColor = $host.UI.RawUI.ForegroundColor,		
 		[System.ConsoleColor]$BackgroundColor = $host.UI.RawUI.BackgroundColor,
 		[ValidateSet('unicode', 'utf7', 'utf8', 'utf32', 'ascii', 'bigendianunicode', 'default', 'oem')]		
 		[string]$Encoding = 'Unicode',
 		[switch]$Force
 	)
 	
 	begin { 		
 		Write-Verbose "Log File: $LogFile"
 		if ( -not $LogFile ) { Write-Error "The -LogFile parameter must be defined or $global:LogFile must be set."; break}		
 		if ( -not (Test-Path $LogFile) ) { New-Item -Path $LogFile -ItemType File -Force | Out-Null }
 		if ( -not (Test-Path $LogFile) ) { Write-Error "Log file can not be found: $LogFile."; break}
 		
 		if ( $Title ) {			
 			$text = $Title
 			$Title = $null
 			Out-LogFile -BlankLine 1 -LogFile $LogFile -WriteHost:$WriteHost -Force:$Force -Encoding $Encoding
 			Out-LogFile -Message $text -BlankLine 1 -DontFormat -LogFile $LogFile -WriteHost:$WriteHost -Force:$Force -Encoding $Encoding 									
 		}
 	}
 	process {
 		
 		if ( $Message ) { 	
 			$text = $Message
 			foreach ( $text in $Message ) {
 				if ( -not $DontFormat ) { $text = "$(($Severity).ToUpper()): $(Get-Date -Format `"$DateFormat`")" + ": $text" }									
 				if ($WriteHost) { Write-Host $text -BackgroundColor $BackgroundColor -ForegroundColor $ForegroundColor}
 				$text | Out-File -FilePath $LogFile -Force:$Force -Encoding $Encoding -Append
 			}		
 		}
 		
 		if ( $BlankLine -gt 0 ){
 			for ($i = 0; $i -lt $BlankLine; $i++ ) { 
 				"" | Out-File -FilePath $LogFile -Force:$Force -Encoding $Encoding -Append
 				if ($WriteHost) { Write-Host "" -BackgroundColor $BackgroundColor -ForegroundColor $ForegroundColor }
 			}
 		}
 	}
 	end {
 	}
 }
 
 $nl = [Environment]::NewLine
 
 $VCCred.Password = ($VCCred.Password| ConvertTo-SecureString -Key $key)
 $VCCred = (New-Object -typename System.Management.Automation.PSCredential -ArgumentList $VCCred.Username,$VCCred.Password)
 
 
 	{ Throw "No datafile supplied, supply path to datafile as an argument to the script! e.g .\SCRIPT.ps1 DATAFILE.txt $nl To create a datafile, the first line of the datafile should be an email address, each subseqenet line should be a VM (one per line)"
 	} 
 else
 	{ $Datafile = $args[0] }
 
 $LogFile = (($DataFile).SubString(0,(($Datafile).length - 3))) + "log"
 $Outfile = (($DataFile).SubString(0,(($Datafile).length - 3))) + "htm"
 $global:WriteHostPreference = $true
 $global:DefaultLogPath =  $LogFile
 
 Out-LogFile -Message "Starting script with $datafile"
 
 if ( (Get-PSSnapin -Name VMware.VimAutomation.Core -ErrorAction SilentlyContinue) -eq $null )
 	{ Add-PsSnapin VMware.VimAutomation.Core }
 
 If (!(Get-PSSnapin -Name VMware.VimAutomation.Core))
 	{Out-LogFile -Message "Failed to load PowerCLI Snap-in. Check PowerCLI is installed." -Severity "E"
 	 Exit(1)
 	}
 
 $HTMLPreString=@"
 <!DOCTYPE html>
 <html lang="en">
 <head>
     <meta charset="UTF-8">
     <link rel="stylesheet" href="http://current.bootstrapcdn.com/bootstrap-v204/css/bootstrap-combined.min.css">
     <style type="text/css">
         body {
             padding: 20px;
         }
 		
 		h2 {
         }
 		
 		.mytable {
         	width: 776px;
         	margin: 12px;
    
         }
 		
 		.alert-info {
 			padding: 8px 8px 8px 14px;
 			margin-top: 12px;
 			margin-left: 12px;
 			margin-right: 12px;
 			margin-bottom: 12px;
 			border: 1px solid;
 			border-radius: 4px;
 		}
 		
 		.chart_wrap {
 			width: 800px;
 			border-style:solid;
 			border-width:1px;
       		margin: 0px 0px 10px 0px;
 		}
 			
 		.chart {
 		 	text-align: center;
 		 	width: 800px;
 		 	height: 400px;
 		}
     </style>
 	<title>
       Virtual Machine Performance statistics
     </title>
     <script type="text/javascript" src="http://www.google.com/jsapi"></script>
     <script type="text/javascript">
       google.load('visualization', '1', {packages: ['corechart']});
     </script>
 
 "@
 $HTMLPreString += "$nl"
 $HTMLBodyBegin  ="<body style=`"font-family: Arial;border: 0 none;`"> $nl" 
 $HTMLBodyBegin  += "<img src=`"$CompanyLogo`" alt=`"Company Logo`" /> $nl"
 $HTMLBodyBegin  += "<h2>Performance Statistics</h2>$nl"
 
 $HTMLPostString=@"
     <script src= "https://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min.js"></script>
     <script src="http://current.bootstrapcdn.com/bootstrap-v204/js/bootstrap.min.js"</script>
 	<script src="http://current.bootstrapcdn.com/bootstrap-v204/js/bootstrap-tab.js"</script>
   </body>
 </html>
 "@
 
 Function Get-DataTable ($Statistics, $UID, $Summation = $false, $Title) 
 {
 $mystring = "<script type=`"text/javascript`">$nl"
 $mystring += "function drawVisualization$UID() {$nl"
 $mystring += "// Create and populate the data table. $nl"
 $mystring += "var " + "$UID" + "data = new google.visualization.DataTable();$nl"
 $mystring += "$UID" + "data.addColumn('datetime', 'Time');$nl"
 $VMs | % {$mystring += "$UID" + "data.addColumn('number', '$_');$nl"}
 $mystring += "$UID" + "data.addRows(" + $Statistics.Count + ");$nl"
 $ColumnCount = 0
 $RowCount = 0
 $Statistics | % {
 	 $ColumnCount = 0
 	 $formatteddate = [datetime]::ParseExact(([string]$_.Name),"M/d/yyyy h:mm:ss tt",$null)
 	 $formatteddate = $formatteddate.ToString("yyyy, $JSMonth, dd, HH, mm")
 	 $formatteddate = "new Date($formatteddate)"
 	 $mystring += "$UID" + "data.setCell($RowCount, $ColumnCount, $formatteddate);$nl"
 	 $_.Group |
 		%  {
 			$ColumnCount = 0
 			foreach ($VM in $VMs)
 				{
 				 $ColumnCount += 1
 				 If ($_.Entity.Name -eq $VM )
 					{ if ($Summation -eq $true)
 						  $strPercent =  [system.math]::round($strPercent,2)
 						  $mystring += "$UID" + "data.setCell($RowCount, $ColumnCount, " + $strPercent + ");$nl"
 						}
 					  else
 						{ $mystring += "$UID" + "data.setCell($RowCount, $ColumnCount, " + $_.Value + ");$nl" }
 					}
 				}
 
 		  	}
 	 $RowCount += 1
 	}
 	$mystring += "$nl new google.visualization.LineChart(document.getElementById('visualization" + "$UID" +"')).$nl"
 	
 	$VisParam = "`
 	{ `
 		legend: {position: 'in',alignment:`"center`"}, `
 		lineWidth:`"2`", `
 		curveType: `"none`",`
 		chartArea:{left:60,top:40,width:`"90%`",height:`"75%`"},`
 		focusTarget:`"category`", `
 		hAxis: {slantedText:`"true`", format:`"E, d MMM`"},`
 		vAxis: {textPosition:`"out`"},`
 		width: 800, `
 		height: 400, `
 		title:`"$Title`"}"
 	
 	$mystring +=  "draw(" + "$UID" + "data, $VisParam);$nl"
 
 	
 	
 	$mystring +="}$nl"
 	$mystring +=  "google.setOnLoadCallback(drawVisualization" + "$UID" + ");$nl</script>$nl"
 	return $mystring
 }
 
 function Get-DivHTML ($UID, $Notes)
 	{
 	$tempHTML = "<div class=`"tab-pane fade`" id=`"$UID`">$nl"
 	$tempHTML += "	<div class=`"chart_wrap`">$nl"
 	$tempHTML += "		<div id=`"visualization" + "$UID" + "`" class=`"chart`"></div>$nl"
 	$tempHTML += "		<div class=`"alert alert-info`"><strong>Information: </strong>$Notes</div>$nl"
 	$tempHTML += "	</div>$nl"
 	$tempHTML += "</div>$nl"
 	return $tempHTML
 	}
 
 
 
 $DataTable = @(Get-Content $Datafile)
 $email =  $DataTable[0] 
 $VMs = @($DataTable[1..($DataTable.Count)])
 Set-PowerCLIConfiguration -InvalidCertificateAction:Ignore -DefaultVIServerMode:Single -Confirm:$false|Out-Null
 	{ Out-LogFile -Message "Failed to connect to vCenter ($VC)" -Severity "E" -WriteHost
 	  Exit (1)
 	}
 else
 	{Out-LogFile -Message "Connected to vCenter ($VC)"}
 
 $VCVMs = (get-vm -name $VMs -ErrorAction SilentlyContinue) 
 $VMs | % {
 		$tmpvm = $_
 		$Exists = $false
 		$VCVMs | % { if ($_.Name -eq $tmpvm) {$Exists = $true}}
 		If ($Exists){
      		 Out-LogFile -Message "$_ found in vCenter inventory"
 			}
 		Else {
      		  Out-LogFile -Message "$_ not found in vCenter inventory" -Severity "W"
 			  $VMs = $VMs |? {$_ -ne $tmpvm}
 			 }
 		}
 
 
 $todayMidnight = (Get-Date -Hour 0 -Minute 0 -Second 0).AddMinutes(-1)
 $metrics = "cpu.usagemhz.average","mem.usage.average","disk.usage.average","net.usage.average","cpu.ready.summation","mem.vmmemctl.average"
 $finish = $todayMidnight
 $startstring = $start.ToString("dddd, dd MMMM yyyy HH:mm")
 $finishstring = $finish.ToString("dddd, dd MMMM yyyy HH:mm")
 Out-LogFile -Message "Getting stats from vCenter"
 Out-LogFile -Message "Got stats from vCenter"
 Out-LogFile -Message "Sorting and filtering stats"
 $CPU = ($Stats | Where-Object {$_.MetricId -eq $metrics[0]} | Sort-Object TimeStamp |Group-Object -Property Timestamp)
 $Memory = ($Stats | Where-Object {$_.MetricId -eq $metrics[1]} | Sort-Object TimeStamp |Group-Object -Property Timestamp)
 $Disk = ($Stats | Where-Object {$_.MetricId -eq $metrics[2]} | Sort-Object TimeStamp |Group-Object -Property Timestamp)
 $Net =  ($Stats | Where-Object {$_.MetricId -eq $metrics[3]} | Sort-Object TimeStamp |Group-Object -Property Timestamp)
 $Ready = ($Stats | Where-Object {$_.MetricId -eq $metrics[4]} | Sort-Object TimeStamp |Group-Object -Property Timestamp)
 $Balloon = ($Stats | Where-Object {$_.MetricId -eq $metrics[5]} | Sort-Object TimeStamp |Group-Object -Property Timestamp)
 
 Out-LogFile -Message "Creating HTML"
 $HTMLOut = $HTMLPreString + (Get-DataTable -Statistics $CPU -UID "CPU" -Title "CPU (MHz)")
 $HTMLOut += (Get-DataTable -Statistics $Memory -UID "Memory" -Title "Memory (%)")
 $HTMLOut += (Get-DataTable -Statistics $Disk -UID "Disk" -Title "Disk Activity (KBps)")
 $HTMLOut += (Get-DataTable -Statistics $Net -UID "Net" -Title "Network I/O (KBps)")
 $HTMLOut += (Get-DataTable -Statistics $Ready -UID "Ready" -Summation $true -Title "CPU Wait Time (%)")
 $HTMLOut += (Get-DataTable -Statistics $Balloon -UID "Balloon" -Summation $true -Title "Memory allocated to the balloon driver (KB)")
 $HTMLOut += "</head>$nl"
 $HTMLOut += $HTMLBodyBegin + $nl
 
 
 $HTMLOut += @"
 
     <div id="content">
         <ul class="nav nav-tabs">
             <li class="dropdown">
               <a href="#" class="dropdown-toggle" data-toggle="dropdown">CPU<b class="caret"></b></a>
                 <ul class="dropdown-menu">
                 </ul>
             </li>
 			<li class="dropdown">
               <a href="#" class="dropdown-toggle" data-toggle="dropdown">Memory<b class="caret"></b></a>
                 <ul class="dropdown-menu">
                 </ul>
 			<li>
         </ul>
         <div id="my-tab-content" class="tab-content">
 			<div class="tab-pane fade in active" id="Info">
 				<div class="chart_wrap">
 					<div class="alert alert-info">
 					<strong>Information: </strong>
 "@
 $HTMLOut += "Each datapoint represents resource consumption since the last datapoint `
 	and its value is either an average, maximum, or summation of all usage in that interval. `
 	Any values that are zero may actually be zero, or may indicate statistics were not `
 	collected over that period (for example, if a virtual machine was powered off). $nl `
 	The statistics displayed are from $startstring to $finishstring</div>$nl"
 $HTMLOut += "<table class=`"mytable table table-striped table-bordered`"><thead><tr><td>VM Name</td><td>Number of CPU's</td><td>Memory (MB)</td><td>CPU Limit</td><td>Memory Limit</td></tr></thead>$nl<tbody>$nl"
 $VCVMs | % { 
 			$tmpname =$_.Name
 			$tmpNumCpu = $_.NumCpu
 			$TmpMemoryMB = $_.MemoryMB
 			$tmpCPULimit = $_.VMResourceConfiguration.CPULimitMhz.ToString().Replace("-1","None")
 			$tmpMemLimit = $_.VMResourceConfiguration.MemLimitMB.ToString().Replace("-1","None")
 			$HTMLOut += "  <tr><td>$tmpname</td><td>$tmpNumCpu</td><td>$TmpMemoryMB</td><td>$tmpMemLimit</td><td>$tmpCPULimit</td></tr>$nl"
 }
 $HTMLOut += "</tbody></table>$nl"
 $HTMLOut +=@"
 		</div>
 	</div>
 			
 "@
 $HTMLOut += (Get-DivHTML -UID "CPU" -Notes "Average CPU usage, as measured in megahertz, during the interval")
 $HTMLOut += (Get-DivHTML -UID "Memory" -Notes "Average memory usage as percentage of total configured or available memory")
 $HTMLOut += (Get-DivHTML -UID "Disk" -Notes "Average disk activity (combinded read & write) in KBps")
 $HTMLOut += (Get-DivHTML -UID "Net" -Notes "Average network utilization (combined transmit and receive rates) during the interval")
 $HTMLOut += (Get-DivHTML -UID "Ready" -Notes "The percentage of time that the virtual machine was ready, but could not get scheduled to run on the physical CPU. Values of less than 10% are considered normal")
 $HTMLOut += (Get-DivHTML -UID "Balloon" -Notes "Amount of memory allocated by the virtual machine memory control driver (vmmemctl), which is installed with VMware Tools. This value should remain at 0 or close to 0")
 $HTMLOut += @"
 			</div>
 		</div>
     </div>
 
 "@
 $datenow = Get-Date -Format "F"
 $HTMLOut += "<h6>Report generated at $datenow<h6>$nl"
 $HTMLOut += $HTMLPostString
 Out-LogFile -Message "Finished building HTML, writing to $Outfile"
 $HTMLOut | Out-File -FilePath $Outfile -Encoding "UTF8"
 
 $body =@"
 <P>Performance statistics for the Virtual Machines listed below are attached</p>
 "@
 $body += "<p>Statistics are from $startstring to $finishstring<p>"
 $VMs | % {$body+= "<li>$_</li>" }
 Out-LogFile -Message "Sending email. Email:$email. SMTP Server:$SMTPServer"
 Send-MailMessage -Attachments $Outfile -Body "$body"  -BodyAsHtml:$true -Subject "Performance Statistics" -To $email -From $SendersAddress -SmtpServer $SMTPServer
 Disconnect-VIServer -Confirm:$false
 Out-LogFile -Message "Disconnected from vCenter"
 Out-LogFile -Message "Finished"
`

