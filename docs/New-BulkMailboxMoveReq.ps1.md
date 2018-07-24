---
Author: dethompson71
Publisher: 
Copyright: 
Email: 
Version: 1.5
Encoding: ascii
License: cc0
PoshCode ID: 4365
Published Date: 2016-08-05t16
Archived Date: 2016-01-30t14
---

# new-bulkmailboxmovereq - 

## Description

move mailboxes from one mailbox database to another mailbox database or balance them across a group of mailbox databases.

## Comments



## Usage



## TODO



## script

`find-archivedbusers`

## Code

`#
 #
 <#
 .SYNOPSIS
 		Move mailboxes from one mailbox database to another mailbox database
 		or balance them across a group of mailbox databases. 
 		Requires Quest AD Tools: 
 		http://www.quest.com/powershell/activeroles-server.aspx
 
 .DESCRIPTION
 		Move mailboxes from one mailbox database to another mailbox database
 		or balance them across a group of mailbox databases.
 
 		Requires Quest AD Tools: 
 		http://www.quest.com/powershell/activeroles-server.aspx
 		
 		Options to move a subset of people based on size. Size is determined
 		by adding TotalItemSize and TotalDeletedItemSize.
 
 		All mailbox moves are created in a suspended state. If you'd like
 		to have the process run, choose the -Monitor switch.
 		
 		You have the choice to work on Primary or Archive mailboxes.
 		
 .PARAMETER SourceDB
 		The mailbox database to used as a source of mailboxes to move.
 			
 .PARAMETER TargetDB
 		The mailbox database to use as a destination for mailboxes to move.
 		If TargetDB is a collection of mailbox databases, 
 		choose the smallest one in the set for each person moved. This takes
 		into account the size of any mailboxes already added.
 		
 		Default is all mailbox databases.
 		
 		You can create a filter in the parameter section, for example:
 		
 		parameter(ValueFromPipeline = $false, Mandatory = $false)] 
 		[array]$TargetDB = (get-mailboxdatabase | ?{$_.Recovery -eq $False -and $-Name -notmatch "Mailbox"})
 
 .PARAMETER Archive
 		The mailboxes to move are Personal Archive mailboxes.
 		
 .PARAMETER Disabled
 		The mailboxes to move are mailboxes for disabled accounts.
 		
 .PARAMETER BatchName
 		A name to identify this group of moves. 
 		Default: The name of the Source Mailbox Database
 			
 .PARAMETER NumberToMove
 		How many mailboxes from the SourceDB to move. When using this
 		parameter, the "select -first $NumberToMove" is used.
 		Default: 1
 
 .PARAMETER MRSServer
 		Specify a particular CAS server to use as this job's 'Mailbox 
 		Replication Server.'
 		
 		This MRSServer will be used for all the moves specified.
 		TargetDB and MRSServer should be in the same AD site.
 		
 .PARAMETER Descending
 		Move the largest mailboxes first. Sorts the mailboxes by size,
 		desending, before selecting the number of mailboxes to move. To
 		move many mailboxes quick the sort is done with the smallest mailbox
 		being the 1st mailbox to move.
 		Default: False
 		
 .PARAMETER SizeLessThan
 		Select the mailbox less than <Number> in size. Number should be in MB
 		
 .PARAMETER RemoveCompletedMoves
 		Before starting a new set of mailbox moves, clear the completed moves 
 		for this batchname.
 		
 .PARAMETER Monitor
 		After the moves have started, monitor their status until all have 
 		completed for this BatchName.
 		
 .PARAMETER MonitorWaitTime
 		Time in seconds to Pause before another monitor cycle.
 		Default: 120
 
 .PARAMETER Notify
 		Send a notice when the Monitor is finished.
 		
 .EXAMPLE
 		New-BulkMailboxMoveRequest -SourceDB 'MBX01A' -TargetDB (get-mailboxdatabase MBX02*)
 	
 		Description
 		-----------	
 		Moves the smallestprimary mailbox from MBX01A to the smallest Mailbox 
 		Database returned in the results of get-mailboxdatabase MBX02*
 
 .EXAMPLE
 		New-BulkMailboxMoveRequest -SourceDB 'Arch01A' -TargetDB (get-mailboxdatabase arch02*) -Desending -Archive
 	
 		Description
 		-----------	
 		Moves the largest archive mailbox from Arch01A to the smallest Mailbox 
 		Database returned in the results of get-mailboxdatabase arch02*	
 		
 .EXAMPLE
 		New-BulkMailboxMoveRequest -SourceDB 'Arch01A' -TargetDB Arch10B -NumberToMove 25 -Archive
 			
 		Description
 		-----------	
 		Moves the smallest 25 archive mailboxes from Arch01A to the Arch10B 
 		Mailbox Database 
 		
 .EXAMPLE
 		New-BulkMailboxMoveRequest -SourceDB 'MBX01A' -TargetDB (get-mailboxdatabase MBX02*) -NumberToMove 25 -SizeLessThan 1000
 			
 		Description
 		-----------	
 		Moves the smallest 25 archive mailboxes from MBX01A to the the collection,
 		but only those under 1GB (1000MB)
 		of mailbox databases returned by (get-mailboxdatabase MBX02*) 
 		The mailboxes are balanced across all the mailbox databases based on Size.
 
 .Notes		Created by Dethompson71 ar gmail dot com
 		http://powershellatwork.blogspot.com/2013/08/new-bulk-mailbox-move-request.html
 		
 		Search for <mydomain> and make your changes, also smtpserver
 
 			
 #>
 
 [CmdletBinding()]
 Param (
 		[parameter(Mandatory = $true, HelpMessage="No Source Mailbox Database name specified")]
 		[ValidateNotNullOrEmpty()]
 		[string]$SourceDB,
 		[parameter(ValueFromPipeline = $false, Mandatory = $false)] 
 		[array]$TargetDB = (get-mailboxdatabase | ?{$_.Recovery -eq $False -and $_.Name -notmatch "Mailbox"}),
 		[switch]$Archive,
 		[switch]$Disabled,
 		[switch]$LitHold,
 		[string]$BatchName = $SourceDB,
 		[int]$NumberToMove	= 1,
 		[string]$MRSServer,
 		[switch]$Descending,
 		[int]$SizeLessThan,
 		[switch]$RemoveCompletedMoves,
 		[switch]$Monitor,
 		[int]$MonitorWaitTime = 120,
 		[switch]$Notify
 )
 Function Find-ArchiveDBUSers {
 	param (
 			[string]$SourceDB,
 			[switch]$Statistics
 	)
 	
 
 
 	$FilteredDB = "(msexcharchivedatabaselink=CN=" + $SourceDB + ",CN=Databases,CN=Exchange Administrative Group (FYDIBOHF23SPDLT),CN=Administrative Groups,CN=<mydomain>,CN=Microsoft Exchange,CN=Services,CN=Configuration,DC=<mydomain>,DC=com)"
 	$user = Get-QADUser -IncludedProperties msexcharchivedatabaselink -SizeLimit 0 -LdapFilter "$FilteredDB"
 	if($Statistics) {
 		$user | %{get-mailboxstatistics $_.mail -archive}
 	} Else {
 		$user | %{get-mailbox $_.mail -archive}
 	}
 }
 #===============================================================================
 if(($Notify) -and (-not ($Monitor))){
 	Write-Verbose "-Notify implies -Monitor; Setting -Monitor..."
 	$Monitor=$True
 }
 
 #===============================================================================
 Write-Verbose "Collecting Mailbox Information in Source Mailbox Database..."
 if ($Archive) {
 	Write-Verbose "Collecting Personal Archive Mailboxes"
 	$SrcMBX = Find-ArchiveDBUSers $SourceDB
 
 	If($SrcMBX){
 		If ($SrcMBX.Count) {
 			"Found {0} Personal Archive mailboxes in {1} database." -f ($SrcMBX.Count),($SourceDB)
 		} Else {
 			"Found 1 Personal Archive mailbox in {0} database." -f ($SourceDB)
 		}
 	} Else {
 		Write-Output "There are no Personal Archive mailboxes to move."
 		Exit
 	}
 } Elseif  ($Disabled) {
 	Write-Verbose "Collecting Disabled Mailboxes"
 	$SrcMBX = get-mailboxdatabase $SourceDB | get-mailbox | ?{$_.ExchangeUserAccountControl -match "AccountDisabled"}
 	If ($SrcMBX.Count) {
 		If ($SrcMBX.Count) {
 			"Found {0} Disabled mailboxes in {1} database." -f ($SrcMBX.Count),($SourceDB)
 		} Else {
 			"Found 1 Disabled mailbox in {0} database." -f ($SourceDB)
 		}
 		
 	} Else {
 		Write-Output "There are no Disabled mailboxes to move."
 		Exit
 	}
 } Elseif  ($LitHold) {
 	Write-Verbose "Collecting Disabled Mailboxes"
 	$SrcMBX = get-mailboxdatabase $SourceDB | get-mailbox | ?{$_.LitigationHoldEnabled}
 
 	If ($SrcMBX.Count) {
 		If ($SrcMBX.Count) {
 			"Found {0} Disabled mailboxes in {1} database." -f ($SrcMBX.Count),($SourceDB)
 		} Else {
 			"Found 1 Disabled mailbox in {0} database." -f ($SourceDB)
 		}
 		
 	} Else {
 		Write-Output "There are no Litigation Hold mailboxes to move."
 		Exit
 	}	
 } Else {
 	Write-Verbose "Collecting Primary Mailboxes"
 	$SrcMBX = get-mailbox -Database $SourceDB -ResultSize Unlimited
 	If($SrcMBX){
 		If ($SrcMBX.Count) {
 			"Found {0} Primary mailboxes in {1} database." -f ($SrcMBX.Count),($SourceDB)
 		} Else {
 			"Found 1 Primary mailbox in {0} database." -f ($SourceDB)
 		}
 	} Else {
 		Write-Output "There are no Primary mailboxes to move."
 		Exit
 	}
 }
 
 
 #===============================================================================
 Write-Verbose "Collecting Destination Mailbox Database Information..."
 $MBDBStats = @()
 $TargetDB | %{
 	$tmp = $null
 	$tmp = Get-Mailboxdatabase ($_) -Status
 	if($tmp){
 		#$tmp | fl
 		$tmpobj = New-Object PSObject
 		$tmpObj | Add-Member -type NoteProperty -name Name -value ($($tmp.AdminDisplayName))
 		#$tmpObj | Add-Member -type NoteProperty -name Size -value ($tmp.DatabaseSize.ToMB() - $tmp.AvailableNewMailboxSpace.ToMB())
 		$tmpObj | Add-Member -type NoteProperty -name Size -value ($tmp.DatabaseSize.ToMB())
 		$MBDBStats += $tmpobj
 	}
 }
 
 If(-not $MBDBStats) {
 	Write-Error "Destination Mailbox Database not found."
 	Exit
 }
 
 #===============================================================================
 Write-Verbose "Collecting Mailbox Statistics..."
 	
 $MBXStats = @()
 if($Archive) {
 	$MBX = $SrcMBX | Get-MailboxStatistics -Archive -EA 0
 } Else {
 	$MBX = $SrcMBX | Get-MailboxStatistics -EA 0
 }
 
 #===============================================================================
 $MBX | %{
 
 	$tmpobj = New-Object PSObject
 	$tmpObj | Add-Member -type NoteProperty -name DisplayName -value ((get-mailbox $_.LegacyDN).DisplayName)
 	$tmpObj | Add-Member -type NoteProperty -name TotalSize -value ($($_.TotalItemSize.Value.ToMB() + $_.TotalDeletedItemSize.Value.ToMB()))
 	$tmpObj | Add-Member -type NoteProperty -name TargetDB -value ("TBD")
 	$MBXStats += $tmpobj
 }
 
 #===============================================================================
 if($SizeLessThan){$MBXStats = $MBXStats | ?{$_.TotalSize -lt $SizeLessThan}}
 $MBXStats = $MBXStats | select -First $NumberToMove
 
 #===============================================================================
 Write-Host "There are $(cc $MBXStats) mailboxes selected."
 if($MBXStats.Count) {
 	for($i=0;$i -lt $MBXStats.Count; ++$i) {
 		$MBDBStats = $MBDBStats | Sort Size	
 		#$MBDBStats | ft
 		$MBXStats[$i].TargetDB = $(($MBDBStats | Select -First 1).Name)
 		#$MBXStats  | ft
 		($MBDBStats | Select -First 1).Size +=  $MBXStats[$i].TotalSize
 	}
 } Else {
 	if ($MBXStats) {
 		$MBDBStats = $MBDBStats | Sort Size	
 		$MBXStats.TargetDB = $(($MBDBStats | Select -First 1).Name)	
 	}
 }
 
 
 #===============================================================================
 If($Descending) {
 	$MBXStats = $MBXStats | sort TotalSize -Descending
 } Else {
 	$MBXStats = $MBXStats | sort TotalSize
 }
 
 If($MBXStats) {
 	Write-Output "Mailboxes to move..."
 	$TotalMoveSize = ($MBXStats | measure -Property TotalSize -Sum).Sum
 	$MBXStats | ft -AutoSize
 
 	Write-Output "`tTotal size of moves: $($TotalMoveSize)`n"
 	$ans = Read-Host "Are you sure you want to continue? (y/n)" 
 	If( -not ($ans.ToUpper() -eq "Y" -or $ans.toUpper() -eq "YES")) { Exit }
 	$StartTime = Get-Date
 	
 } Else {
 	Write-Output "The commandline options did not select any mailboxes for moving."
 	Exit
 }
 
 #===============================================================================
 if($RemoveCompletedMoves) {
 	Write-Verbose "Trying to remove Completed Requests..."
 	if($BatchName) {
 		Get-MoveRequest -BatchName $BatchName -MoveStatus Completed | 
 		Remove-MoveRequest -Confirm:$false
 	}
 }
 
 
 #===============================================================================
 #
 ForEach ($MBXStat in $MBXStats) {
 	Write-Verbose "Destination Mailbox Database Count: $($MBDBStats.Count)"
 	Write-Verbose "Working on $($MBXStat.DisplayName)"
 			
 	$mbx = get-mailbox $MBXStat.DisplayName
 	Write-verbose "$($mbx.DisplayName) size $($MBXStat.TotalSize) moving to $($MBXStat.TargetDB)"
 	
 	if ($Archive) {
 		if ($MRSServer) {
 	    	$mbx | New-MoveRequest -ArchiveOnly -ArchiveTargetDatabase $MBXStat.TargetDB  -BatchName $BatchName -Suspend -MRSServer $MRSServer -BadItemLimit 49
 		} Else {
 			$mbx | New-MoveRequest -ArchiveOnly -ArchiveTargetDatabase $MBXStat.TargetDB -BatchName $BatchName -Suspend -BadItemLimit 49
 		}
 	} else {
 		if ($MRSServer) {
 	    	$mbx | New-MoveRequest -PrimaryOnly -TargetDatabase $MBXStat.TargetDB -BatchName $BatchName -Suspend -MRSServer $MRSServer -BadItemLimit 49
 		} Else {
 			$mbx | New-MoveRequest -PrimaryOnly -TargetDatabase $MBXStat.TargetDB -BatchName $BatchName -Suspend -BadItemLimit 49
 		}
 	}
 	
 	
 }
 
 
 if($Monitor) {
 	#===============================================================================
 
 	Write-Verbose "Resuming Mailbox Moves..."
 	Sleep -seconds 10
 	Get-MoveRequest -BatchName $BatchName -MoveStatus 'Suspended' | Resume-MoveRequest 
 	Write-Verbose "Starting Mailbox Move Monitoring..."
 	Do {
 		Write-Verbose "Collecting Move Stats..."
 		$stats = Get-MoveRequest -BatchName $BatchName | Get-MoveRequestStatistics
 		$stats = $stats | Sort DisplayName
 		$out = "`n{0,-30}{1,16}{2,16}{3,10}" -f "DisplayName","TargetDB","PercentComplete","Size (MB)"
 		Write-host $out
 		$out = "{0,-30}{1,16}{2,16}{3,10}"   -f "-----------","-------------","---------------","---------"
 		Write-Host $out
 		foreach ($Stat in $stats) {
 			$out = "{0,-30}{1,16}{2,16}{3,10}" -f ($stat.Displayname), ($stat.TargetArchiveDatabase),($stat.PercentComplete),($stat.TotalMailboxSize.ToMB())
 			Write-Host $out
 		}
 		Write-Host "`n"
 		$CurTime = (Get-Date) - ($StartTime)
 		$out = "{0}`tElasped Time: {1}h {2}m {3}s" -f (Get-Date -Format 'MMM dd yyyy HH:mm'), ($CurTime.Hours),($CurTime.Minutes),($CurTime.Seconds)
 		Write-Host $out
 		Write-Verbose "Time to snooze..."
 	    sleep -seconds $MonitorWaitTime
 	} While ((Get-MoveRequest -BatchName $BatchName | ?{$_.Status -ne 'Completed' -and $_.Status -ne 'Failed'}))
 	
 	
 	if ($Notify) {
 		$Subject  = "Mailbox Database Bulk Move Complete: $BatchName"
 		$To       = 'admin@<mydomain>.com'
 		$From	  = 'BulkMoveRequest@<mydomain>.com'
 		$Body     =  $(Get-MoveRequest -BatchName $BatchName) | Out-String
 		Send-MailMessage -Body $Body -To $To -From $From -SmtpServer 'smtpserver.<mydomain>.com' -Subject $Subject #-BodyAsHtml
 	}
 	
 }
`

