---
Author: alanrenouf
Publisher: 
Copyright: 
Email: 
Version: 1.12
Encoding: ascii
License: cc0
PoshCode ID: 1213
Published Date: 
Archived Date: 2009-07-22t10
---

# vmware daily report - 

## Description

for more information and sample out put please visit http

## Comments



## Usage



## TODO



## function

`send-smtpmail`

## Code

`#
 #
 param( [string] $VISRV)
 
 #
 $SMTPSRV = "myexchangeserver.mydomain.comk"
 #
 $EmailFrom = "reports@mydomain.com"
 #
 $EmailTo = "myemail@mydomain.com"
 
 $SetUsername = ""
 $CredFile = ".\mycred.crd"
 $DatastoreSpace = "5"
 $SnapshotAge = 14
 $VMsNewRemovedAge = 5
 $VCEventAge = 1
 $VCEvntlgAge = 1
 
 
 #######################################
 if ($VISRV -eq ""){
 	Write-Host
 	Write-Host "Please specify a VI Server name eg...."
 	Write-Host "      powershell.exe DailyReport.ps1 MYVISERVER"
 	Write-Host
 	Write-Host
 	exit
 }
 
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
 
 <meta name="save" content="history">
 
 <style type="text/css">
 TABLE {TABLE-LAYOUT: fixed; FONT-SIZE: 100%; WIDTH: 100%}
 *{margin:0}
 .dspcont1{ display:none}
 a.dsphead1 span.dspchar{font-family:monospace;font-weight:normal;}
 td {VERTICAL-ALIGN: TOP; FONT-FAMILY: Tahoma}
 BODY {margin-left: 4pt} 
 BODY {margin-right: 4pt} 
 BODY {margin-top: 6pt} 
 </style>
 </head>
 <body>
 <font face="Arial" size="1"><b>Generated on $($ENV:Computername)</b></font><br>
 <font face="Arial" size="1">Report created on $(Get-Date)</font>
 <div class="filler"></div>
 <div class="filler"></div>
 <div class="filler"></div>
 <div class="save">
 "@
 Return $Report
 }
 
 Function Get-CustomHeader0 ($Title){
 $Report = @"
 		<h1><a class="dsphead0">$($Title)</a></h1>
 	<div class="filler"></div>
 "@
 Return $Report
 }
 
 Function Get-CustomHeader ($Num, $Title){
 $Report = @"
 	<h2><a class="dsphead$($Num)">
 	$($Title)</a></h2>
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
 	Return $HTMLTable
 }
 
 Function Get-HTMLDetail ($Heading, $Detail){
 $Report = @"
 <TABLE>
 	<tr>
 	<th width='25%'><b>$Heading</b></font></th>
 	<td width='75%'>$($Detail)</td>
 	</tr>
 </TABLE>
 "@
 Return $Report
 }
 
 function Find-Username ($username){
 	if ($username -ne $null)
 	{
 		$root = [ADSI]""
 		$filter = ("(&(objectCategory=user)(samAccountName=$Username))")
 		$ds = new-object  system.DirectoryServices.DirectorySearcher($root,$filter)
 		$ds.PageSize = 1000
 		$ds.FindOne()
 	}
 }
 
 function Get-VIServices
 {
 	If ($SetUsername -ne ""){
 		$Services = get-wmiobject win32_service -Credential $creds -ComputerName $VISRV | Where {$_.DisplayName -like "VMware*" }
 	} Else {
 		$Services = get-wmiobject win32_service -ComputerName $VISRV | Where {$_.DisplayName -like "VMware*" }
 	}
 	
 	$myCol = @()
 
 	Foreach ($service in $Services){
 		$MyDetails = "" | select-Object Name, State, StartMode, Health
 		If ($service.StartMode -eq "Auto")
 		{
 			if ($service.State -eq "Stopped")
 			{
 				$MyDetails.Name = $service.Displayname
 				$MyDetails.State = $service.State
 				$MyDetails.StartMode = $service.StartMode
 				$MyDetails.Health = "Unexpected State"
 			}
 		}
 		If ($service.StartMode -eq "Auto")
 		{
 			if ($service.State -eq "Running")
 			{
 				$MyDetails.Name = $service.Displayname
 				$MyDetails.State = $service.State
 				$MyDetails.StartMode = $service.StartMode
 				$MyDetails.Health = "OK"
 			}
 		}
 		If ($service.StartMode -eq "Disabled")
 		{
 			If ($service.State -eq "Running")
 			{
 				$MyDetails.Name = $service.Displayname
 				$MyDetails.State = $service.State
 				$MyDetails.StartMode = $service.StartMode
 				$MyDetails.Health = "Unexpected State"
 			}
 		}
 		If ($service.StartMode -eq "Disabled")
 		{
 			if ($service.State -eq "Stopped")
 			{
 				$MyDetails.Name = $service.Displayname
 				$MyDetails.State = $service.State
 				$MyDetails.StartMode = $service.StartMode
 				$MyDetails.Health = "OK"
 			}
 		}
 		$myCol += $MyDetails
 	}
 	Write-Output $myCol
 }
 
 function Get-DatastoreSummary {
 	param(
 		$InputObject = $null
 	)
 	begin {
 	}
 	process {
 		if ($InputObject -and $_) {
 			throw 'The input object cannot be bound to any parameters for the command either because the command does not take pipeline input or the input and its properties do not match any of the parameters that take pipeline input.'
 			return
 		}
 		$processObject = $(if ($InputObject) {$InputObject} else {$_})
 		if ($processObject) {
 			$myCol = @()
 			foreach ($ds in $_)
 			{
 				$MyDetails = "" | select-Object Name, Type, CapacityMB, FreeSpaceMB, PercFreeSpace
 				$MyDetails.Name = $ds.Name
 				$MyDetails.Type = $ds.Type
 				$MyDetails.CapacityMB = $ds.CapacityMB
 				$MyDetails.FreeSpaceMB = $ds.FreeSpaceMB
 				$MyDetails.PercFreeSpace = [math]::Round(((100 * ($ds.FreeSpaceMB)) / ($ds.CapacityMB)),0)
 				$myCol += $MyDetails
 			}
 			$myCol | Where { $_.PercFreeSpace -lt $DatastoreSpace }
 		}
 	}
 	end {
 	}
 }
 
 function Get-SnapshotSummary {
 	param(
 		$InputObject = $null
 	)
 
 	BEGIN {
 	}
 
 	PROCESS {
 		if ($InputObject -and $_) {
 			throw 'ParameterBinderStrings\AmbiguousParameterSet'
 			break
 		} elseif ($InputObject) {
 			$InputObject
 		} elseif ($_) {
 			
 			$mySnaps = @()
 			foreach ($snap in $_){
 				$SnapshotInfo = Get-SnapshotExtra $snap
 				$mySnaps += $SnapshotInfo
 			}
 
 			$mySnaps | Select VM, Name, @{N="DaysOld";E={((Get-Date) - $_.Created).Days}}, @{N="Creator";E={(Find-Username (($_.Creator.split("\"))[1])).Properties.displayname}}, Created, Description -ErrorAction SilentlyContinue | Sort DaysOld
 
 		} else {
 			throw 'ParameterBinderStrings\InputObjectNotBound'
 		}
 	}
 
 	END {
 	}
 }
 
 function Get-SnapshotTree{
 	param($tree, $target)
 	
 	$found = $null
 	foreach($elem in $tree){
 		if($elem.Snapshot.Value -eq $target.Value){
 			$found = $elem
 			continue
 		}
 	}
 	if($found -eq $null -and $elem.ChildSnapshotList -ne $null){
 		$found = Get-SnapshotTree $elem.ChildSnapshotList $target
 	}
 	
 	return $found
 }
 
 function Get-SnapshotExtra ($snap){
 	$taskMgr = Get-View TaskManager
 	
 	$report = @{}
 	
 	$filter = New-Object VMware.Vim.TaskFilterSpec
 	$filter.Time = New-Object VMware.Vim.TaskFilterSpecByTime
 	$filter.Time.beginTime = (($snap.Created).AddDays(-5))
 	$filter.Time.timeType = "startedTime"
 	
 	$collectionImpl = Get-View ($taskMgr.CreateCollectorForTasks($filter))
 	
 	$dummy = $collectionImpl.RewindCollector
 	$collection = $collectionImpl.ReadNextTasks($tasknumber)
 	while($collection -ne $null){
 		$collection | where {$_.DescriptionId -eq "VirtualMachine.createSnapshot" -and $_.State -eq "success" -and $_.EntityName -eq $guestName} | %{
 			$row = New-Object PsObject
 			$row | Add-Member -MemberType NoteProperty -Name User -Value $_.Reason.UserName
 			$vm = Get-View $_.Entity
 			if($vm -ne $null){ 
 				$snapshot = Get-SnapshotTree $vm.Snapshot.RootSnapshotList $_.Result
 				if($snapshot -ne $null){
 					$key = $_.EntityName + "&" + ($snapshot.CreateTime.ToString())
 					$report[$key] = $row
 				}
 			}
 		}
 		$collection = $collectionImpl.ReadNextTasks($tasknumber)
 	}
 	$collectionImpl.DestroyCollector()
 	
 	$snapshotsExtra = $snap | % {
 		$key = $_.vm.Name + "&" + ($_.Created.ToString())
 		if($report.ContainsKey($key)){
 			$_ | Add-Member -MemberType NoteProperty -Name Creator -Value $report[$key].User
 		}
 		$_
 	}
 	$snapshotsExtra
 }
 
 Function Set-Cred ($File) {
 	$Credential = Get-Credential
 	$credential.Password | ConvertFrom-SecureString | Set-Content $File
 }
 
 Function Get-Cred ($User,$File) {
 	$password = Get-Content $File | ConvertTo-SecureString
 	$credential = New-Object System.Management.Automation.PsCredential($user,$password)
 	$credential
 }
 
 If ($SetUsername -ne ""){
 	if ((Test-Path -Path $CredFile) -eq $false) {
 		Set-Cred $CredFile
 	}
 	$creds = Get-Cred $SetUsername $CredFile
 }
 
 $VIServer = Connect-VIServer $VISRV
 If ($VIServer.IsConnected -ne $true){
 	$USER = $env:username
 	$APPPATH = "C:\Documents and Settings\" + $USER + "\Application Data"
 
 	if ($env:appdata -eq $null -or $env:appdata -eq 0)
 	{
 		$env:appdata = $APPPATH
 	}
 	$VIServer = Connect-VIServer $VISRV
 	If ($VIServer.IsConnected -ne $true){
 		send-SMTPmail -to $EmailTo -from $EmailFrom -subject "ERROR: $VISRV Daily Report" -smtpserver $SMTPSRV -body "The Connect-VISERVER Cmdlet did not work, please check you VI Server."
 		exit
 	}
 	
 }
 
 
 $VM = Get-VM
 $VMH = Get-VMHost
 $Clusters = Get-Cluster
 $Datastores = Get-Datastore
 $FullVM = Get-View -ViewType VirtualMachine
 
 $MyReport = Get-CustomHTML "$VIServer Daily Report"
 	$MyReport += Get-CustomHeader0 ($VIServer.Name)
 		
 		$MyReport += Get-CustomHeader "1" "General Details"
 			$MyReport += Get-HTMLDetail "Number of Hosts:" (($VMH).Count)
 			$MyReport += Get-HTMLDetail "Number of VMs:" (($VM).Count)
 			$MyReport += Get-HTMLDetail "Number of Clusters:" (($Clusters).Count)
 			$MyReport += Get-HTMLDetail "Number of Datastores:" (($Datastores).Count)
 		$MyReport += Get-CustomHeaderClose
 		
 		$Snapshots = $VM | Get-Snapshot | Where {$_.Created -lt ((Get-Date).AddDays(-$SnapshotAge))} | Get-SnapshotSummary
 		If (($Snapshots | Measure-Object).count -gt 0) {
 			$MyReport += Get-CustomHeader "1" "Snapshots (Over $SnapshotAge Days Old)"
 				$MyReport += Get-HTMLTable $Snapshots
 			$MyReport += Get-CustomHeaderClose
 		}
 				
 		$OutputDatastores = $Datastores | Get-DatastoreSummary | Sort PercFreeSpace
 		If (($OutputDatastores | Measure-Object).count -gt 0) {
 			$MyReport += Get-CustomHeader "1" "Datastores (Less than $DatastoreSpace% Free)"
 				$MyReport += Get-HTMLTable $OutputDatastores
 			$MyReport += Get-CustomHeaderClose
 		}
 		
 		$MaintHosts = $VMH | where {$_.State -match "Maintenance"} | Select name,CustomFields
 		If (($MaintHosts | Measure-Object).count -gt 0) {
 			$MyReport += Get-CustomHeader "1" "Hosts in Maintenance Mode"
 				$MyReport += Get-HTMLTable $MaintHosts
 			$MyReport += Get-CustomHeaderClose
 		}
 		
 		$RespondHosts = $VMH | where {$_.State -match "Not"} | Select name,CustomFields
 		If (($RespondHosts | Measure-Object).count -gt 0) {
 			$MyReport += Get-CustomHeader "1" "Hosts not responding"
 				$MyReport += Get-HTMLTable $RespondHosts
 			$MyReport += Get-CustomHeaderClose
 		}
 		
 		$VIEvent = Get-VIEvent -maxsamples 10000 -Start (Get-Date).AddDays(-$VMsNewRemovedAge)
 		$OutputCreatedVMs = $VIEvent | where {$_.Gettype().Name -eq "VmCreatedEvent" -or $_.Gettype().Name -eq "VmBeingClonedEvent" -or $_.Gettype().Name -eq "VmBeingDeployedEvent"} | Select createdTime, @{N="User";E={(Find-Username (($_.userName.split("\"))[1])).Properties.displayname}}, fullFormattedMessage
 		If (($OutputCreatedVMs | Measure-Object).count -gt 0) {
 			$MyReport += Get-CustomHeader "1" "VMs Created or Cloned (Last $VMsNewRemovedAge Day(s))"
 				$MyReport += Get-HTMLTable $OutputCreatedVMs
 			$MyReport += Get-CustomHeaderClose
 		}
 		
 		$OutputRemovedVMs = $VIEvent | where {$_.Gettype().Name -eq "VmRemovedEvent"}| Select createdTime, @{N="User";E={(Find-Username (($_.userName.split("\"))[1])).Properties.displayname}}, fullFormattedMessage
 		If (($OutputRemovedVMs | Measure-Object).count -gt 0) {
 			$MyReport += Get-CustomHeader "1" "VMs Removed (Last $VMsNewRemovedAge Day(s))"
 				$MyReport += Get-HTMLTable $OutputRemovedVMs
 			$MyReport += Get-CustomHeaderClose
 		}
 		
 		$OutputErrors = Get-VIEvent -maxsamples 10000 -Start (Get-Date).AddDays(-$VCEventAge ) -Type Error | Select createdTime, @{N="User";E={(Find-Username (($_.userName.split("\"))[1])).Properties.displayname}}, fullFormattedMessage
 		If (($OutputErrors | Measure-Object).count -gt 0) {
 			$MyReport += Get-CustomHeader "1" "Error Events (Last $VCEventAge Day(s))"
 				$MyReport += Get-HTMLTable $OutputErrors
 			$MyReport += Get-CustomHeaderClose	
 		}
 		
 		$NoTools = $FullVM | Where { $_.Runtime.PowerState -eq "poweredOn" } | Select Name, @{N="ToolsVersion"; E={$_.Config.tools.toolsVersion}} | Where { $_.ToolsVersion -eq 0} | Select Name
 		If (($NoTools | Measure-Object).count -gt 0) {
 			$MyReport += Get-CustomHeader "1" "No VMTools"
 				$MyReport += Get-HTMLTable $NoTools
 			$MyReport += Get-CustomHeaderClose
 		}
 		
 		$CDConn = $VM | Where { $_ | Get-CDDrive | Where { $_.ConnectionState.Connected -eq "true" } } | Select Name, Host
 		If (($CDConn | Measure-Object).count -gt 0) {
 			$MyReport += Get-CustomHeader "1" "VM: CD-ROM Connected - VMotion Violation"
 				$MyReport += Get-HTMLTable $CDConn
 			$MyReport += Get-CustomHeaderClose
 		}
 		
 		$Floppy = $VM | Where { $_ |  Get-FloppyDrive | Where { $_.ConnectionState.Connected -eq "true" } } | Select Name, Host
 		If (($Floppy | Measure-Object).count -gt 0) {
 			$MyReport += Get-CustomHeader "1" "VM:Floppy Drive Connected - VMotion Violation"
 				$MyReport += Get-HTMLTable $Floppy
 			$MyReport += Get-CustomHeaderClose
 		}
 		
 		$MyReport += Get-CustomHeader "1" "$VIServer Service Details"
 			$MyReport += Get-HTMLTable (Get-VIServices)
 		$MyReport += Get-CustomHeaderClose
 		
 		$ConvDate = [System.Management.ManagementDateTimeConverter]::ToDmtfDateTime([DateTime]::Now.AddDays(-$VCEvntlgAge))
 		If ($SetUsername -ne ""){
 			$ErrLogs = Get-WmiObject -Credential $creds -computer $VIServer -query ("Select * from Win32_NTLogEvent Where Type='Error' and TimeWritten >='" + $ConvDate + "'") | Where {$_.Message -like "*VMware*"} | Select @{N="TimeGenerated";E={$_.ConvertToDateTime($_.TimeGenerated)}}, Message 
 		} Else {
 			$ErrLogs = Get-WmiObject -computer $VIServer -query ("Select * from Win32_NTLogEvent Where Type='Error' and TimeWritten >='" + $ConvDate + "'") | Where {$_.Message -like "*VMware*"} | Select @{N="TimeGenerated";E={$_.ConvertToDateTime($_.TimeGenerated)}}, Message
 		}
 		
 		If (($ErrLogs | Measure-Object).count -gt 0) {
 			$MyReport += Get-CustomHeader "1" "$VIServer Event Logs: Error"
 				$MyReport += Get-HTMLTable ($ErrLogs)
 			$MyReport += Get-CustomHeaderClose
 		}
 		
 		$ConvDate = [System.Management.ManagementDateTimeConverter]::ToDmtfDateTime([DateTime]::Now.AddDays(-1))
 		If ($SetUsername -ne ""){
 			$WarnLogs = Get-WmiObject -Credential $creds -computer $VIServer -query ("Select * from Win32_NTLogEvent Where Type='Warning' and TimeWritten >='" + $ConvDate + "'") | Where {$_.Message -like "*VMware*"} | Select @{N="TimeGenerated";E={$_.ConvertToDateTime($_.TimeGenerated)}}, Message 
 		} Else {
 			$WarnLogs = Get-WmiObject -computer $VIServer -query ("Select * from Win32_NTLogEvent Where Type='Warning' and TimeWritten >='" + $ConvDate + "'") | Where {$_.Message -like "*VMware*"} | Select @{N="TimeGenerated";E={$_.ConvertToDateTime($_.TimeGenerated)}}, Message 
 		}
 		If (($WarnLogs | Measure-Object).count -gt 0) {
 			$MyReport += Get-CustomHeader "1" "$VIServer Event Logs: Warning"
 				$MyReport += Get-HTMLTable ($WarnLogs)
 			$MyReport += Get-CustomHeaderClose
 		}
 			
 	$MyReport += Get-CustomHeader0Close
 $MyReport += Get-CustomHTMLClose
 
 #$Date = Get-Date
 #$Filename = "C:\Temp\" + $VIServer + "DailyReport" + "_" + $Date.Day + "-" + $Date.Month + "-" + $Date.Year + ".htm"
 #$MyReport | out-file -encoding ASCII -filepath $Filename
 
 send-SMTPmail $EmailTo $EmailFrom "$VISRV Daily Report" $SMTPSRV $MyReport
 
 $VIServer | Disconnect-VIServer -Confirm:$false
`

