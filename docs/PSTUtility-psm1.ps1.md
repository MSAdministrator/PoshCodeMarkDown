---
Author: dethompson71
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3810
Published Date: 2014-12-03t09
Archived Date: 2014-03-11t22
---

# pstutility.psm1 - 

## Description

this file was too large, i guess i need to post it some where else.

## Comments



## Usage



## TODO



## script

`new-pstfilelogobject`

## Code

`#
 #
 #=============================================================================
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #=============================================================================
 
 
 
 #=============================================================================
 #=============================================================================
 
 #====> TO DO -- add your servername and drive location
 
 
 $Script:PSTQueueFile		= $Script:PSTShareDir + '\PSTImportJobQueue.csv'
 
 
 #=============================================================================
 #=============================================================================
 
 $Script:LocalADSite         = 'HQ' 
 
 $Script:PSTUtilityVersion 	= "14.1.20121015"
 
 
 #=============================================================================
 $Script:SpecialPeople = @("<DisplayName1>","<Displayname2>")
 
 #=============================================================================
 $Script:IgnoreUsers   = @("<DisplayName1>","<DisplayName2>") 
 
 
 #=============================================================================
 #=============================================================================
 
 Function New-PSTFileLogObject(){
 <#
 .SYNOPSIS
 	Return a new object to track a PST file.
 
 .DESCRIPTION
 	Return a new object to track a PST file.
 	This is created for each new PST discovered for any user
 	
 #>
 	$PSTObj = New-Object PSObject
 	$PSTObj | Add-Member -type NoteProperty -name UNCFullPathName			-value (0)
 	$PSTObj | Add-Member -type NoteProperty -name LastWriteTime				-value ("")
 	$PSTObj | Add-Member -type NoteProperty -name Size						-value (0)
 	$PSTObj | Add-Member -type NoteProperty -name IgnoreFile				-value ("FALSE")
 	$PSTObj | Add-Member -type NoteProperty -name DateRemoved				-value ("")
 	$PSTObj | Add-Member -type NoteProperty -name BackupFolksNotified		-value ("FALSE")
 	$PSTObj | Add-Member -type NoteProperty -name AgeInDays					-value (0)
 	
 	$PSTObj
 }
 Function Get-ArchPSTIndex ($PST=$null) {
 <#
 .SYNOPSIS
 	Creates or finds an index number for PST name.
 	
 .DESCRIPTION
 	Creates or finds an index number for PST name.
 	Uses the full UNC path for index building
 
 #>
 	
 	$Return = $null`
 	
 	If ($Script:PSTIndex.ContainsKey($PST.VersionInfo.FileName)) {
 		$Return = $PSTIndex.Item($PST.VersionInfo.FileName)
 	} Else {
 		
 		$PSTLog = New-ArchPSTFileLogObject
 		$PSTLog.UNCFullPathName = $PST.VersionInfo.FileName
 		$Script:ArchPSTQueue += $PSTLog
 		$Script:PSTIndex.Add($PSTLog.UNCFullPathName,$Script:ArchPSTQueue.Count)
 		$Return = $Script:ArchPSTQueue.Count
 	}
 	
 	$Return
 	
 }
 Function New-ArchPSTFileLogObject(){
 <#
 .SYNOPSIS
 	Return a new object to track PST file.
 
 .DESCRIPTION
 	Return a new object to track a PST file.
 	This is created for each new PST discovered for any users with an Archive Mailbox
 	
 #>
 	$PSTObj = New-Object PSObject
 	$PSTObj | Add-Member -type NoteProperty -name UNCFullPathName			-value (0)
 	$PSTObj | Add-Member -type NoteProperty -name LastWriteTime				-value ("")
 	$PSTObj | Add-Member -type NoteProperty -name Size						-value (0)
 	$PSTObj | Add-Member -type NoteProperty -name IgnoreFile				-value ("FALSE")
 	$PSTObj | Add-Member -type NoteProperty -name DateChecked				-value ("")
 	$PSTObj | Add-Member -type NoteProperty -name BackupFolksNotified		-value ("FALSE")
 	$PSTObj | Add-Member -type NoteProperty -name AgeInDays					-value (0)
 	$PSTObj | Add-Member -type NoteProperty -name AcctName					-value ("")
 	$PSTObj | Add-Member -type NoteProperty -name Dept						-value ("")
 	$PSTObj | Add-Member -type NoteProperty -name Server					-value ("")
 	$PSTObj | Add-Member -type NoteProperty -name DateDiscovered			-value (Get-date)
 	$PSTObj | Add-Member -type NoteProperty -name DateCreated				-value ("")
 		
 	$PSTObj
 
 }
 Function Get-PSTFileIndex ($PST=$null) {
 <#
 .SYNOPSIS
 	Creates or finds an index number for PST name.
 	
 .DESCRIPTION
 	Creates or finds an index number for PST name.
 	Uses the full UNC path for index building
 
 #>
 	
 	$Return = $null`
 	
 	Write-Host "Enter [Get-PSTFileIndex]: $(($PST).VersionInfo.FileName) Count: $(($Script:AllPSTQueue).Count)"
 	
 	If ($Script:PSTIndex.ContainsKey($PST.VersionInfo.FileName)) {
 		$Return = $Script:PSTIndex.Item($PST.VersionInfo.FileName)
 		Write-Host "Found [Get-PSTFileIndex]: $($Return)"
 	} Else {
 		
 		$PSTLog = New-PSTFileLogObject
 		$PSTLog.UNCFullPathName = $PST.VersionInfo.FileName
 		Write-Host "Fullname $(($PSTLog).UNCFullPathName)"
 		
 		Write-Host "Before: " $PSTLog
  		
 		$Script:AllPSTQueue += $PSTLog
 		Write-host "After: " $Script:AllPSTQueue[-1]
 		
 		#$Script:PSTIndex.Add($_.UNCFullPathName,$i)
 		$Script:PSTIndex.Add($PSTLog.UNCFullPathName,($Script:AllPSTQueue).Count-1)
 		Write-Host "Added? : " $Script:PSTIndex.Get_item($PSTLog.UNCFullPathName)
 		$Return = ($Script:AllPSTQueue).Count -1
 		
 		Write-host "Try to reindex in: " $Script:AllPSTQueue[($Return)]
 		Write-Host "Add [Get-PSTFileIndex]: $($Return)"
 		
 	}
 	Write-Host "Leave [Get-PSTFileIndex]"
 	$Return
 	
 	
 }
 Function New-PSTJobObject() {
 <#
 .SYNOPSIS
 	Object for storing information about a PST Import Job.
 
 .DESCRIPTION
 	Object for storing information about a PST Import Job.
 	A user many have many Jobs, one for each PST file being imported.
 #>
 
 	$PSTJob = New-Object PSObject
 	
 	
 	
 	
 	$PSTJob | Add-Member -type NoteProperty -name JobCreationTime	-value ($null)	
 	
 	
 	$PSTJob
 }
 Function New-PSTReportObj (){
 <#
 .SYNOPSIS
 	PST Report Object for tracking the progress of PST migration.
 
 .DESCRIPTION
 	PST Report Object for tracking the progress of users with 
 	an Archive Mailbox and their progress at removing PST files 
 	from their home drives.
 	
 
 #>
 
 	$ReportObj = New-Object PSObject
 	
 	$ReportObj | Add-Member -type NoteProperty -name DisplayName		-value ("")
 	
 	$ReportObj | Add-Member -type NoteProperty -name InUsePST			-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name HDrvPST			-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name HDrvSize			-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name ImpPST				-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name ImpSize			-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name TotalSize			-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name SkippedPST			-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name DaysSinceImport	-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name GPO				-value ("")
 	$ReportObj | Add-Member -type NoteProperty -name '-'				-value ("")
 	
 	$ReportObj | Add-Member -type NoteProperty -name MBXDB				-value ("")
 	$ReportObj | Add-Member -type NoteProperty -name ArchDB				-value ("")
 	$ReportObj | Add-Member -type NoteProperty -name MBXSize			-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name MBXQuota			-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name ArchSize			-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name ArchQuota			-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name InitArchSize		-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name ArchGrowth			-value (0)
 	$ReportObj | Add-Member -type NoteProperty -name DateCreated		-value ((Get-Date))
 	
 	
 		
 	$ReportObj
 }
 Function Reset-PSTBackupFileNames () {
 <#
 .SYNOPSIS
 	Reset Backup file names to current time
 .DESCRIPTION
 	Reset them all so that you'll be guaranteed to have the correct
 	name each time
 #>
 	
 	$Script:ImportQueueFileBU  	= $Script:PSTShareDir + '\ZZSavedQueueLogs\PSTImportJobQueue' + "-" + $Script:TE + ".csv"
 	$Script:CompleteQueueFileBU	= $Script:PSTShareDir + '\ZZSavedQueueLogs\PSTImportCompleteQueue' + "-" + $Script:TE + ".csv"
 	$Script:ArchReportFileBU   	= $Script:PSTUNCDir   + '\ZZSavedQueueLogs\ArchiveReport-' + $Script:TE + '.csv'	
 	$Script:ArchPSTFileBU     	= $Script:PSTUNCDir   + '\ZZSavedQueueLogs\ArchivePSTHistory-' + $Script:TE + '.csv'
 	$Script:AllPSTFileLogBU		= $Script:PSTUNCDir   + '\ZZSavedQueueLogs\AllPSTHistory-' + $Script:TE + '.csv' 
 
 
 
 }
 
 #=============================================================================
 #=============================================================================
 Function CC () {
 <#
 .SYNOPSIS
 	a utility to return the count of members in a collection
 .DESCRIPTION
 	a utility to return the count of members in a collection
 	0, 1, or many ...
 #>
 	
 	Param (
 		$Col
 	)
 	if ($Col) {
 		if ($Col.Count -eq $null) {
 			$Cnt = 1
 		} Else {
 			$Cnt = $Col.Count
 		}
 	} Else {	
 		$Cnt = 0	
 	}
 	$Cnt
 }
 Function isMemberOf() {
 <#
 .SYNOPSIS
 	A utility to check if a mailbox is a member of a group.
 
 .DESCRIPTION
 	A utility to check if a mailbox is a member of a group.
 #>	
 	Param (
 		[string] $groupName,
 	) 
 	$m = Get-QADGroupMember $groupName -SizeLimit 0 | where { $_.Displayname -eq $memberName }
 	if($m) {$true} else {$false}
 	
 }
 Function WhoAmI () {
 <#
 .SYNOPSIS
 	sometimes you want to know who is running this job ;)
 
 .DESCRIPTION
 	ometimes you want to know who is running this job ;)
 #>
 	
 	([System.Security.Principal.WindowsIdentity]::GetCurrent()).Name
 	
 }
 Function Get-ISODayOfWeek {
 <#
 .SYNOPSIS
 	Get the correct (according to ISO 8601) day of the week
 
 .DESCRIPTION
 	For any given date you get the day of the week, according to ISO8610
 	
 .PARAMETER Date
 	The date you want to analyze. Accepts dates on the pipeline
 
 #>
 	Param([DateTime]$date=$(Get-Date))
 	Process {
 	  if($_){$date=$_}
 	  @(7,1,2,3,4,5,6)[$date.DayOfWeek]
 	}
 }
 Function Get-FreeDiskSpace($drive,$computer) {
 <#
 .SYNOPSIS
 	Utility to discover the free space of a drive
 
 .DESCRIPTION
 	Utility to discover the free space of a drive
 	Used to decide if there is enough space to copy a PST file
 	to either the Temp processing area, or to the local PC for backup.
 #>
 	
  	$driveData = Get-WmiObject -class win32_LogicalDisk -computername $computer -filter "Name = '$drive'"
 	#"$computer free disk space on drive $drive"
 	"{0:n2}" -f ($driveData.FreeSpace/1MB)
 	
 }
 Function ConvertTo-ComputerName() {
 <#
 .SYNOPSIS
  	Resolve the IP to a computer name
 
 .DESCRIPTION
 	CAS logs record IP address for a user, but those can change over time
 	resolve the IP to a computer name to save in the PST Job Queue
 	- try to look up in DNS
 	- 90% of time this will be an IP
 	
 #>    
 	
 	Param (
 		$FindName
 	)
 	
     $Return = $null
 	$ErrorActionPreference = "SilentlyContinue"
     $c = [System.Net.Dns]::GetHostbyAddress($FindName).HostName
 	$ErrorActionPreference = "Continue"
 	
 	if ($c) {
     	$Return = ($c.Split(".")[0]).ToUpper()
     } Else {
 		$Return = $FindName
 	}
 	$Return
 }
 Function ConvertTo-IP () {
 <#
 .SYNOPSIS
 	Lookup IP using the ComputerName
 
 .DESCRIPTION
 	CAS logs record IP address for a user, but those can change over time
 	resolve the IP to a computer name to save in the PST Job Queue
 #>	
     Param ($FindName)
 	
     $Return = $null
 	$ErrorActionPreference = "SilentlyContinue"
 	$c = [System.Net.Dns]::GetHostEntry($FindName).Addresslist[0].ToString()
 	$ErrorActionPreference = "Continue"
 	
 	if ($c) {
     	$Return = ($c.Split(".")[0]).ToUpper()
     } Else {
 		$Return = $FindName
 	}
 	$Return
 }
 Function Clean-OnlinePSTGPO(){
 <#
 .SYNOPSIS
 	Clear up GPO group and make sure it has only the correct members
 
 .DESCRIPTION
 	Clear up GPO group and make sure it has only the correct members
 
 .Notes
 	In our world, there are 2 types of PST file users: Controled and Non-Controlled
 	Controled Users:
 		New users created after a certain date (1FEB2012) and Users migrated to
 		Online PSTs
 	Non-Controlled Users:
 		Those users created befor 1FEB2012 and not having been migrated.
 	
 	Eventually the Non-Controlled user will disappear. At least, we can hope.
 #>
 
 	$GPO = get-group 'GPO-ReadOnlyPST' | select -ExpandProperty Members
 
 	$GPO | %{
 		
 		Write-Host "Working " $_.ToString()
 		$Dumpable = $false
 		$Reason = "None"
 		$MBX = get-mailbox $_.ToString()
 				
 		if(! (($MBX).DataBase.Name)) {
 			$Dumpable = $true
 			$Reason = "NoMailbox"
 		}
 		Elseif (!(($MBX).ArchiveDataBase.Name)) {
 			if ((get-date '2/1/2012') -gt (get-date ($MBX).WhenMailboxCreated)){
 				$Dumpable = $true
 				$Reason = "No archive and 2/1/2012 is greater than " + $(($MBX).WhenMailboxCreated)
 			}
 		}
 		
 		
 		if ($Dumpable){
 			Write-Host "Removing: " ($MBX).Displayname " - " $Reason
 			Remove-QADGroupMember -Identity 'GPO-ReadONlyPST' -Member $_.ToString()
 			
 		}
 	}
 		
 	
 }
 
 
 #=============================================================================
 #=============================================================================
 Function Add-ToGPO ($User = $null) {
 <#
 .SYNOPSIS
 	Add user to a group that defines scope of GPO to Disallow PST Growth
 
 .DESCRIPTION
 	In our organization our GPO that disallows PST growth is governed by a
 	group during the transition, later we can apply that setting worldwide.
 	
 	For users to be comfortable, they need to Read their old PST files
 
 	These skipped users are legal discovery people that need to continue to
 	create PSTs as they gather info for Lawyer types. 
 		
 #>	
 	
 	if ($User) {
 		if (-not ($Script:IgnoreUser -contains (get-mailbox $User).SamAccountName)) {
 			Add-QADGroupMember -Identity 'gpo-ReadonlyPST' -Member (get-mailbox $User).SamAccountName
 		}
 	}
 	
 }
 Function Adjust-Quota ($User = $null) {
 <#
 .SYNOPSIS
 	Make sure people are not hit by a quota during their transition.
 
 .DESCRIPTION
 	Make sure people are not hit by a quota during their transition Set new 
 	quota to 500MB -- unless theie quota is already huge
 #>
 	
 	if ($User) {
 		$MBX = get-mailbox $User
 		$UsingDBQuotas = $MBX.UseDatabaseQuotaDefaults
 		if ($UsingDBQuotas -eq $True){ 
 			$Database = Get-MailboxDatabase -Identity $MBX.Database
 			$ProhibitSendQuota = $Database.ProhibitSendQuota.value.ToMB() -as [Int]
 			$IssueWarningQuota = $Database.IssueWarningQuota.value.ToMB() -as [Int]
 		
 		} Else {
 			$ProhibitSendQuota = $MBX.ProhibitSendQuota.value.ToMB() -as [Int]
 			$IssueWarningQuota = $MBX.IssueWarningQuota.value.ToMB() -as [Int]
 		}
 		
 		$NewProhibitSendQuota = 500 -as [int]
 		$NewIssueWarningQuota = 450 -as [int]
 		
 		if($NewIssueWarningQuota -gt $IssueWarningQuota) {
 			Set-Mailbox -Identity $MBX -UseDatabaseQuotaDefaults:$False -ProhibitSendQuota 500MB -IssueWarningQuota 450MB -ProhibitSendReceiveQuota "Unlimited"
 		}
 	}
 }
 Function Get-ClientAccessUserLog () {
 <#
 .SYNOPSIS
 	Discover a user's Client and IP information.
 	
 .DESCRIPTION
 	Discover a user's client and IP information. Script uses this function too.
 	
 	There is also nightly process that runs which gets all the "connect" log 
 	entries for OUTLOOK.EXE. It's run overnight. Initially we were looking at all
 	the logs and that was very teadious. Now we search an already created subset.
 		
 	It's best to pass in the LegacyExchangeDN and SAMAccountName from the mailbox object
 	from testing it seems to get better results right now using both
 	
 	There is no way of knowing where this user logged in so we must search
 	all CAS servers until we find a "connect" entry and pass the full line back
 	
 	Also users can log in from many PCs -- we only want the most used
 	so look thru the full day and get all info you can, then group by IP and 
 	take the one with the most connects.
 	
 	alas, sometimes you won't find anything ...
 	
 
 #>
 	param (
 		$DisplayName = $null,
 		$SamName = $null, 
 		$LegacyName = $null, 
 		$IP = $null,
 		$SearchDays = 14,
 		[switch]$Quiet
 	)
 	
 	#-----------------------------------------------------------------
 	
 	$Results = @()
 	If($DisplayName) {
 		$MB = Get-SingleMailboxObject $DisplayName 
 		$LegacyName = $MB.LegacyExchangeDN
 		$SamName    = $MB.SamAccountName
 	} Elseif ($IP) {
 		$LegacyName = $IP
 	}
 		
 	$Days = -1
 	Do {
 		$Y  = Get-Date $((get-date).adddays($Days)) -Format yyyyMMdd
 		if (!($Quiet)) {Write-Host "Searching "$Y}
 		if(Test-path ($Script:CASConnectLogDir + "\olConnect-"+$Y+"*")) {
 			$tmpResults = gc (gci ($Script:CASConnectLogDir + "\olConnect-"+$Y+"*")) | ?{($_ -match ($LegacyName + ",")) -and ($_ -match ",,Outlook") -and ($_ -match "Connect,")}
 			if($tmpResults) { 
 				$tmpResults | %{$Results += $_}
 			}
 		}
 		$Days--
 	} While (!($Results) -And ($Days -gt ($SearchDays * (-1))))
 	
 	if ($Results) {
 		$c = $Results | %{"{0};{1};{2};{3}" -f $_.Split(",")[3],$_.Split(",")[6],$_.Split(",")[8],$_.Split(",")[7]}
 		$d = $c | %{if($_.split(";")[2]){$_.Split(";")[2]}} | group -NoElement | sort -Descending
 		if($d){
 			$e = $c | ?{$_ -match $d.Values}
 		}
 		
 		if     ($e.count)   {
 			$e[0]
 		} Elseif ($e)         {
 			$e
 		} Else                {
 			if ($c.count)   {
 				$c[0]
 			} Else {
 				$c
 			}
 		}
 	}
 }
 Function Get-MRServer ($MBX = $null) {
 <#
 .SYNOPSIS
 	Pick an MRSServer in the same AD Site as the destination MBX Database.
 
 .DESCRIPTION
 	If you are using deticated CAS servers to import the PST files use this function
 	to pick the correct one. Pick the wrong MRServer and the Import will never
 	start - here I pick the an MR Server in the Same AD Site using the CAS as a
 	guide we have dediticated "pst import" CAS servers.
 	
 	We are chosing the One CAS server we have designated to heap many request 
 	upon so we don't effect user's normal performance on production CAS servers.
 
 .NOTES
 	We have since moved all our live copies of each Archive Mailboxdatabase to the
 	same AD site.
 	
 	It is also possible to skip this step, and let exchange pick a cas server.
 	Depends on how much you are moving and how you feel that will effect your users
 	See "Import-PSTFromShare" 
 #>
 	#---------------------------------------------------------------------
 
 	$Return = $null	
 	
 	
 	
 	
 	$Return = "cas02.domain.com"
 	
 	$Return
 	
 }
 Function Get-OutlookEXEVersion () {
 <#
 .SYNOPSIS
 	Discover the version of Outlook on a PC
 
 .DESCRIPTION
 	Discover the version of Outlook on a PC
 	
 	There are many version and many people do not have a 
 	compatible version with Exchange 2010 Archive Mailbox
 	
 	
 #>
 	param(
 		$ComputerName = $null
 	)
 	
 	$OL = $Null
 				
 	if($ComputerName) {
 		if (Test-Connection -ComputerName $ComputerName -Quiet -Count 1) {
 			Write-Verbose "Trying $ComputerName"
 			
 				Write-Verbose "Office 2010 on 64bit"
 				$OL = gci ('\\' + $ComputerName + '\c$\Program Files (x86)\Microsoft Office\Office14\outlook.exe')
 				Write-Verbose "Office 2010"
 				$OL = gci ('\\' + $ComputerName + '\c$\Program Files\Microsoft Office\Office14\outlook.exe')
 				Write-Verbose "Office 2007"
 				$OL = gci ('\\' + $ComputerName + '\c$\Program Files\Microsoft Office\Office12\outlook.exe')
 			} Else {	
 				if (Test-Path ('\\' + $ComputerName + '\c$\Program Files\Microsoft Office\OFFICE11\outlook.exe')) {
 					Write-Verbose "Office 2003"
 					$OL = gci ('\\' + $ComputerName + '\c$\Program Files\Microsoft Office\OFFICE11\outlook.exe')
 				} Else {
 					Write-Verbose "Can't determine Outlook version [$ComputerName]"	
 				}
 			} 
 		} Else {
 			Write-Warning "[$ComputerName] is not responding"
 		}
 	} Else {
 		Write-Verbose "missing computername [$ComputerName]"	
 	}
 	
 	$OL.VersionInfo.ProductVersion
 }
 Function Get-SingleMailboxObject($Name = $null) {
 <#
 .SYNOPSIS
 	Return a single mailbox
 
 .DESCRIPTION
 	Since we are importing mail and humans can be error prone,
 	make absolutely sure we only get one hit on the requested mailbox
 
 #>
 	#-----------------------------------------------------------------
 		
 	$ReturnMBX = $null	
 	if ($Name) {
 		$MBX = get-mailbox $Name -EA 0
 			}
 		}
 	}
 	$ReturnMBX
 }
 Function New-OnlineArchiveByJob($JobObj = $null) {
 <#
 .SYNOPSIS
 	Based on Job information, give user Online PST feature.
 
 .DESCRIPTION
 	If this job's user is not enabled for online archive
 	then create one in the smallest database.
 	
 	The smallest DB was determined during the "add-PSTImportQueue"
 	($JobObj.TargetDB)
 
 #>	
 
 	$Return = $false
 	
 	if ($JobObj) {
 		$MBXObj = Get-SingleMailboxObject $JobObj.UserEmail
 		
 		
 		Enable-Mailbox -Archive -Identity $mbxObj -ArchiveDatabase ($JobObj.TargetDB) -ArchiveName $('Online PST - ' + ($mbxObj).Displayname) -ea 0
 		sleep -Seconds 5
 		
 		Adjust-Quota $MBXObj
 		
 		$MBXObj = Get-SingleMailboxObject $JobObj.UserEmail
 		If ($MBXObj.ArchiveDatabase) {
 			$Return = $true
 		} Else {
 			$Return = $false
 		}
 	}
 	$Return
 }
 Function Test-OutlookClientVersion ($V = $null) 	{
 <#
 .SYNOPSIS
 	Test Oulook.exe version
 .DESCRIPTION
 	older outlook clients can't see the "Online Archive"
 	so flag those users who need an upgrade
 	
 	12.0.6550.5000 is the minimum level - buggy
 	12.0.6607.xxxx is Office 2007 SP3 --  preferred level
 		there are search bugs in earlier versions
 	
 	14.0.6109.5005 is outlook 14 patched to Nov 8, 2011
 	14.0.6117.5xxx is patched thru April 2012 -- preferred
 
 #>
 	$VOK = $False
 	if($V) {
 		
 		if ($V.Split(".")[0] -eq 14) {
 			if ($V.Split(".")[2] -ge 4760) {$VOK = $true}
 			
 		} Elseif ($V.Split(".")[0] -eq 12) {
 			if ($V.Split(".")[2] -ge 6550) {$VOK = $true}
 			
 		}
 	}
 	$VOK
 }
 
 
 #=============================================================================
 #=============================================================================
 Function Copy-PSTtoImportShare($Path=$Null, $Dest=$null){
 <#
 .SYNOPSIS
 	copy the pst to our share folder
 
 .DESCRIPTION
 	Copy the pst to our share folder using Copy-item in the same local AD site
 	works fine. Using BITS when copying over WAN lines
 #>	
 	
 	if($Path -and $Dest ) {
 	
 		$NewFile = Copy-Item -Path $Path -Destination $Dest -PassThru
 		if ($NewFile){
 			$Result = "Copied"
 		} Else {
 			$Result = $error[0].Exception
 			Write-Verbose "Copy ResultFrom: $($error[0].Exception)"
 		}
 	}
 	$Result
 	
 }
 Function Copy-PSTusingBITS ($JobObj = $null) {
 <#
 .SYNOPSIS
 	copy the pst to our share folder
 
 .DESCRIPTION
 	Copy the pst to our share folder using Copy-item in the same local AD site
 	works fine. Using BITS when copying over WAN lines
 #>	
 	$Result = $false
 	
 	if ($JobObj) {
 		
 		$params = @{
 		    Source = ($Jobobj.OrgUNCName)
 		    Destination = ($Jobobj.TargetUNCName)
 		    Description = ("Online PST BITS Copy")
 		    DisplayName = ($JobObj.JobName)
 		    Asynchronous = $true
 			Priority = "Normal"
 		} 
 
 		$Result = Start-BitsTransfer @params
 		
 	}
 	$Result = $null
 	sleep 2
 	$Result = Get-BitsTransfer | Where-Object { $_.DisplayName -eq ($ThisJob.JobName) }
 	$Result
 }
 Function Get-PSTFileList() {
 <#
 .SYNOPSIS
 	Discover PST Files in folders
 	
 .DESCRIPTION
 	Discover PST Files in folders
 	
 .NOTES	
 	- Some AD objects do not have a home directory
 	- Allowed for some results of *.pst being directories
 
 #>
 	param(
 		$Homedir = $null, 
 		$PCIP = $null, 
 		$SearchPC = $false, 
 		$OS = $null, 
 		$UN = $null, 
 		$SourceDir=$null
 	)
 	
 	$HDFiles = $null
 	$PCFIles = $null
 	
 	
 	
 	if(!($SearchPC) -and $SourceDir) {
 			Write-host "`t-> Override HomeDir: $HomeDir to $SourceDir"
 			$Homedir = $SourceDir
 	}
 	If ($Homedir){
 		if(Test-Path $HomeDir) {
 			Write-Verbose "`t`t-> Searching $HomeDir"
 			[array]$HDFiles = gci -path $HomeDir -Filter '*.pst' -Recurse -Force -EA 0 | ?{$_.PSIsContainer -eq $false}
 		}
 	}
 	
 	
 	if($SearchPC -and $PCIP) {
 		if(Test-Connection $PCIP -Count 1 -Quiet) {
 			
 			$PCDir = $null
 			if($SourceDir) {
 				$PCDir = $SourceDir
 			} Else {			
 				if($OS -match "7") {
 					Write-Verbose "`t`t-> Searching Windows7 PC $UN"
 					$PCDir = $("\\" + $PCIP + "\C$\Users\" + $UN + "\AppData\Local\Microsoft\Outlook")
 					
 				} Elseif ($OS -match "XP") {
 					Write-Verbose "`t`t-> Searching WindowsXP PC $UN"
 					$PCDir = $("\\" + $PCIP + "\C$\Documents And Settings\" + $UN)
 				} Else {
 					Write-Verbose "`t`t-> OS not known, testing for OS :"
 					if(Test-Path ("\\" + $PCIP + "\C$\Users\" + $UN  + "\AppData\Local\Microsoft\Outlook")) {
 						Write-Verbose "`t`t-> Try Windows7"
 						$PCDir = "\\" + $PCIP + "\C$\Users\" + $UN  + "\AppData\Local\Microsoft\Outlook"
 					} Elseif (Test-Path ("\\" + $PCIP + "\C$\Documents And Settings\" + $UN)){
 						Write-Verbose "`t`t-> Try WindowsXP"
 						$PCDir = $("\\" + $PCIP + "\C$\Documents And Settings\" + $UN)
 					} Else {
 						Write-Verbose "`t`t-> OS Unknown"
 						$PCDir = $("\\" + $PCIP + "\c$")
 					}
 				}
 			}
 			
 			
 			Write-Verbose "`t`t-> Searching $PCDir"
 			If(Test-Path $PCDir) {
 				[array]$PCFIles = gci -path ($PCDir) -Filter '*.pst' -Recurse -Force -EA 0 | ?{$_.PSIsContainer -eq $false}
 			}
 			
 		} Else {
 			Write-Output ("`t-> $PCIP is not responding.")
 		}
 	}
 	if($HDFiles -and $PCFIles){
 		$Files = $HDFiles + $PCFIles
 	} Elseif ($HDFiles -and (-not $PCFIles)){
 		$Files = $HDFiles
 	} Elseif ((-not $HDFiles) -and $PCFIles){
 		$Files = $PCFIles
 	} Else {
 		$Files = $null 
 	}
 	
 	$Files
 }
 Function Import-PSTFromShare($JobObj){
 <#
 .SYNOPSIS
 	import into the Archive mailbox
 
 .DESCRIPTION
 	import into the Archive mailbox
 	
 .NOTES
 	We found that 99% of the people wanted to keep their data separate after the
 	import, just like it was in their PST files, so UseTargetRootFolder was 
 	defaulted to true in the NEw-PSTJobObject function
 	
 	We check here for false.
 #>
 
 	$Return		= "Failed"
 	$Error.Clear | Out-Null	
 	$MBXName	= $JobObj.UserObj
 	$User       = $JobObj.UserObj.Split("\")[1]
 	$File 		= $JobObj.TargetUNCName
 	$JobName	= $JobObj.JobName
 	$CAS		= $JobObj.MRServer
 	if ($JobObj.TargetRootFolder) {
 		$Folder = $JobObj.TargetRootFolder
 	} Else {
 		$Folder = $JobObj.OrgFileName
 	}
 	
 	Write-Debug "$MBXName -Name  $JobName  -Batchname  $USER  -MRSServer  $CAS -TargetDB   $JobObj.TargetDB"
 	
 	#======= TO DO ============
 	
 	if($JobObj.UseTargetRootFolder -eq "TRUE") {
 		$PSTQueued = New-MailboxImportRequest -Name $JobName -BatchName $USER -Mailbox "$MBXName" -FilePath "$File" -IsArchive -MRSServer $CAS -BadItemLimit 9999 -AcceptLargeDataLoss -Confirm:$false -targetrootfolder $Folder
 	} Else {
 		$PSTQueued = New-MailboxImportRequest -Name $JobName -BatchName $USER -Mailbox "$MBXName" -FilePath "$File" -IsArchive -MRSServer $CAS -BadItemLimit 9999 -AcceptLargeDataLoss -Confirm:$false
 	}
 	If($PSTQueued) {
 		$Return = $PSTQueued.Status
 	} Else { 
 		$Return = $Error[0].ToString()
 	}
 	sleep -Seconds 5
 	$Return
 }
 Function Move-PSTToLocal ($IP=$null, $PSTFiles=$null, $SamAcct=$null, $IgnoreAge=$False) {
 <#
 .SYNOPSIS
 	clear PST files off the Home shares
 	
 .DESCRIPTION
 	The botton line of this project is to clear PST files off the Home shares
 	If the PST file has been imported the file itself has become a backup
 	and as such is safe to put on the local PC of the user.
 	
 .NOTES
 	Move PSTs
 	If this is a windows 7 box and has 2010 this might be the place to put them
 	  \\172.18.20.1\c$\Users\<user>\Documents\Outlook Files	
 	
 	but this is good for XP	
 	  \\172.18.20.1\c$\D9ocuments and Settings\<user>\My Docuemtns\Outlok Files
 	
 	If this user is in a far away place, then do NOT push the data back to the PC
 
 #>	
 	
 	
 	Write-Host "Starting = Who:($SamAcct), Count:($(($PSTFiles).Count)), Computer:($IP)" 
 	
 	$HQUser = $true
 	$Region = (get-mailbox $SamAcct).CustomAttribute2
 	If (($Region -eq "Asia") -or ($Region -eq "Europe")) {$HQUser = $false}
 	if ($IP -eq "None") {$IP = $null}
 	
 	if($IP -and $PSTFIles -and $SamAcct -and $HQUser) {
 		Write-Host $("$SamAcct; Starting Move-PSTFiles $(get-date)")
 		$("$SamAcct; Starting Move-PSTFiles $(get-date)") >> $Script:MoveFileLog
 		if (Test-Connection -ComputerName $IP -Quiet){
 			
 			Write-Host $("$SamAcct; Connect to $IP passed")
 			$("$SamAcct; Connect to $IP passed") >> $Script:MoveFileLog
 			
 			$AvailableSpace = [int](get-freediskspace 'c:' $IP)/2
 			$PSTFiles		= $PSTFiles | Sort Length
 			$PSTSize        = (($PSTFIles | Measure-Object Length -sum).Sum)/(1024*1024)
 			$OldPSTSize		= ((($PSTFIles | ?{$_.LastWriteTime -le (get-date ((get-date).adddays(-90))) } | Measure-Object Length -sum).Sum)/(1024*1024))
 	
 			$Continue = $false
 			if (Test-Path $('\\' + $IP + '\c$\users\' + $SamAcct)) {
 				
 				$LocalBU = $('\\' + $IP + '\c$\users\' + $SamAcct+ '\Documents\Outlook Files')
 				if (!(Test-Path $LocalBU)){ mkdir $LocalBU }
 				$Continue = $true
 				Write-Host $("$SamAcct; Connect to $IP passed")
 				$("$SamAcct; LocalBU is $LocalBU") >> $Script:MoveFileLog
 				
 			} Elseif (Test-Path $('\\' + $IP + '\c$\documents and settings\' + $SamAcct)) {
 			
 				$LocalBU = $('\\' + $IP + '\c$\documents and settings\' + $SamAcct + '\Outlook Files')
 				if (!(Test-Path $LocalBU)){ mkdir $LocalBU }
 				$Continue = $true
 				Write-Host $("$SamAcct; LocalBU is $LocalBU")
 				$("$SamAcct; LocalBU is $LocalBU") >> $Script:MoveFileLog
 				
 			} Else {
 				Write-Host "Can not find a secure directory for $SamAcct"
 				$Continue = $false
 				$("$SamAcct; LocalBU is Can not find a secure directory; Fail") >> $Script:MoveFileLog
 				
 			}
 			
 			
 			
 			if ($Continue) {	
 				$FileNames = @()
 				$PSTFiles | %{
 					$ThisPST = $_
 					$FN = $ThisPST.Name
 					$PSTBackups = $LocalBU
 					$FileAge = (((get-date) - ($ThisPST.LastWriteTime)).Days)
 					
 					Write-Host "Working: $FN"
 					Write-Host "(($AvailableSpace) -gt ($PSTSize))"
 					Write-Host "IgnoreAge is $IgnoreAge"
 					if($AvailableSpace -gt $PSTSize) {
 						
 						If($IgnoreAge) {
 							if($FileNames -contains $FN -or (Test-Path $($PSTBackups + "\" + $FN))){
 								$rn = "{0:00}" -f (Get-Random -maximum 10000)
 								[string]$FN = $ThisPST.Name.Split(".")[0] + "-" + $RN  + "." + ($ThisPST.Name.Split(".")[1])
 								$PSTBackups = $PSTBackups + "\" + $FN
 								$("$SamAcct; Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)") >> $Script:MoveFileLog
 								Write-Host "Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)" 
 								Move-Item -Path $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)
 							} Else {
 								$PSTBackups = $PSTBackups + "\" + $FN
 								$("$SamAcct; Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)") >> $Script:MoveFileLog
 								Write-Host "Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)" 
 								Move-Item -Path $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)
 							}
 						} Else {
 							If ($ThisPST.LastWriteTime -le (get-date ((get-date).adddays(-30)))){
 
 								if($FileNames -contains $FN -or (Test-Path $($PSTBackups + "\" + $FN))){
 									$rn = "{0:00}" -f (Get-Random -maximum 10000)
 									[string]$FN = $ThisPST.Name.Split(".")[0] + "-" + $RN  + "." + ($ThisPST.Name.Split(".")[1])
 									$PSTBackups = $PSTBackups + "\" + $FN
 									$("$SamAcct; Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)") >> $Script:MoveFileLog
 									Write-Host "Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)" 
 									Move-Item -Path $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)
 								} Else {
 									$PSTBackups = $PSTBackups + "\" + $FN
 									$("$SamAcct;Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)") >> $Script:MoveFileLog
 									Write-Host "Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)" 
 									Move-Item -Path $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)
 								}
 							} Else {
 								$("$SamAcct; Age [$($FileAge)] Skipping due to Age<30 $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)") >> $Script:MoveFileLog
 								Write-Host "Age [$($FileAge)] Skipping due to Age<30  $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)"
 							}
 						}
 					} Else {
 						
 						$AvailableSpace = [int](get-freediskspace 'c:' $IP)/2
 						$PSTSize        = (($ThisPST | Measure-Object Length -sum).Sum)/(1024*1024)
 						If($AvailableSpace -gt $PSTSize) {
 							Write-Host "(($AvailableSpace) -gt ($PSTSize))"
 							If($IgnoreAge) {
 												
 									if($FileNames -contains $FN -or (Test-Path $($PSTBackups + "\" + $FN))){
 										$rn = "{0:00}" -f (Get-Random -maximum 10000)
 										[string]$FN = $ThisPST.Name.Split(".")[0] + "-" + $RN  + "." + ($ThisPST.Name.Split(".")[1])
 										$PSTBackups = $PSTBackups + "\" + $FN
 										$("$SamAcct; Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)") >> $Script:MoveFileLog
 										Write-Host "Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)" 
 										Move-Item -Path $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)
 									} Else {
 										$PSTBackups = $PSTBackups + "\" + $FN
 										$("$SamAcct; Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)") >> $Script:MoveFileLog
 										Write-Host "Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)" 
 										Move-Item -Path $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)
 									}
 							} Else {
 								If ($ThisPST.LastWriteTime -le (get-date ((get-date).adddays(-30)))){
 
 									if($FileNames -contains $FN -or (Test-Path $($PSTBackups + "\" + $FN))){
 										$rn = "{0:00}" -f (Get-Random -maximum 10000)
 										[string]$FN = $ThisPST.Name.Split(".")[0] + "-" + $RN  + "." + ($ThisPST.Name.Split(".")[1])
 										$PSTBackups = $PSTBackups + "\" + $FN
 										$("$SamAcct; Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)") >> $Script:MoveFileLog
 										Write-Host "Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)" 
 										Move-Item -Path $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)
 									} Else {
 										$PSTBackups = $PSTBackups + "\" + $FN
 										$("$SamAcct; Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)") >> $Script:MoveFileLog
 										Write-Host "Age [$($FileAge)] Moving $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)" 
 										Move-Item -Path $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)
 									}
 								} Else {
 									$("$SamAcct; Age [$($FileAge)] Skipping due to Age<30 $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)") >> $Script:MoveFileLog
 								}
 							}
 						} Else {
 							$("$SamAcct; Age [$($FileAge)] Skipping due to AvailableSpace $($ThisPST.VersionInfo.Filename) -Destination  $($PSTBackups)") >> $Script:MoveFileLog
 						}
 					}
 					$FileNames += $FN
 				}
 				
 			} Else {
 				Write-Host "No Files Moved."
 				$("$SamAcct; No Files Moved.") >> $Script:MoveFileLog
 			}
 		} Else {
 			Write-Host "Computer [$IP] seems dead"
 			$("$SamAcct; Computer [$IP] seems dead.") >> $Script:MoveFileLog
 		}
 	} Else {
 		if (!($PSTFiles)) {
 			Write-Host "No PST Files found."
 			$("$SamAcct; No PST Files found.") >> $Script:MoveFileLog
 		} Else {
 				Write-Host "Bad Parameters or Not CONUS User."
 			$("$SamAcct; Bad Parameters or Not CONUS User.") >> $Script:MoveFileLog
 		}
 	}
 }
 
 
 #=============================================================================
 #=============================================================================
 Function Format-JobStatus($JobObj) {
 <#
 .SYNOPSIS
 	Query the queue for info on a particular user.
 .DESCRIPTION
 	Query the queue for info on a particular user.
 	
 	Count the number of jobs in each status category, incoming JobObj is a 
 	collection.
 #>
 	#
 	#
 	
 	$JobsCount       = 0
 	$StatusNew       = 0
 	$StatusCopied    = 0
 	$StatusBITS      = 0
 	$StatusInQueue   = 0
 	$StatusImported  = 0
 	$StatusCleanedup = 0
 	$StatusNotified  = 0
 	$LastError       = $null
 	$JobSize         = 0
 	$StatusSkipped	 = 0
 	
 	$JobObj | %{
 	
 		$ClientOK = $_.ClientVerOK
 		$UName =  $_.UserMBX
 		$TargetDB = $_.TargetDB
 		$a = [int]($_.FileSize)
 		$JobSize += $a
 		$skip=$false
 		if(($_.ProcessFile -eq "FALSE")){$Skip=$true}
 		
 		
 		Switch ($_.JobStatus) {
 			New {
 				if(($Skip)){$StatusSkipped++} Else {$StatusNew++}
 			}
 			NewBits {
 				if(($Skip)){$StatusSkipped++} Else {$StatusNew++}
 			}
 			Copied {$StatusCopied++}
 			InBits {$StatusBITS++}
 			InQueue {$StatusInQueue++}
 			Imported {$StatusImported++}
 			CleanedUp {$StatusCleanedup++}
 			Notified {$StatusNotified++}
 		}
 		$JobsCount++
 	}
 	$StatusEntry = New-Object PSObject
 	$StatusEntry | Add-Member -type NoteProperty -name Name      	-value ($UName)
 	$StatusEntry | Add-Member -type NoteProperty -name Client    	-value ($ClientOK)
 	$StatusEntry | Add-Member -type NoteProperty -name Size		   	-value ($JobSize)
 	$StatusEntry | Add-Member -type NoteProperty -name Jobs      	-value ($JobsCount)
 	$StatusEntry | Add-Member -type NoteProperty -name New      	-value ($StatusNew)
 	$StatusEntry | Add-Member -type NoteProperty -name Skip      	-value ($StatusSkipped)
 	$StatusEntry | Add-Member -type NoteProperty -name BITS      	-value ($StatusBITS)
 	$StatusEntry | Add-Member -type NoteProperty -name Copied      	-value ($StatusCopied)
 	$StatusEntry | Add-Member -type NoteProperty -name InQue     	-value ($StatusInQueue)
 	$StatusEntry | Add-Member -type NoteProperty -name Imptd    	-value ($StatusImported)
 	$StatusEntry | Add-Member -type NoteProperty -name ClndUp   	-value ($StatusCleanedup)
 	$StatusEntry | Add-Member -type NoteProperty -name Notifd 	   	-value ($StatusNotified)
 	$StatusEntry | Add-Member -type NoteProperty -name Target 	   	-value ($TargetDB)
 	#$StatusEntry | Add-Member -type NoteProperty -name Errors      	-value ($LastError)
 	
 	$StatusEntry
 }
 Function Get-ArchDB() {
 <#
 .SYNOPSIS
 	Find the smallest Archive mailbox database
 .DESCRIPTION
 	Find the smallest Archive mailbox database, not necesarily the smallest now, 
 	but even after all the current jobs are processed
 #>
 	
 	$ArchSize = @{}
 	(get-mailboxdatabase |?{$_.name -match 'arch\d{3}[acdef]' -and $_.name -notmatch 'arch102'} | Get-MailboxDatabase -Status) | %{$ArchSize.Add($_.Name, $_.DatabaseSize.ToMB())}
 	
 	if($OldJobQueue) {
 		$OldJobQueue | %{
 			$tmp = $ArchSize.item($_.TargetDB)
 			$Tmp += [int]$_.FileSize
 			$ArchSize.remove($_.TargetDB)
 			$ArchSize.add($_.TargetDB, $tmp)
 		}
 	}
 	
 	($ArchSize.GetEnumerator() | sort Value)[0].Name
 }
 Function Lock-PSTIQ() {
 <#
 .SYNOPSIS
 	Signal the queue file is in use by creating a file.
 
 .DESCRIPTION
 	Signal the queue file is in use by creating a file.
 	
 #>
 	if(!(Test-PSTIQLock)) {$a = New-Item $($Script:PSTUNCDir + '\PSTImportQueue.Lock') -itemtype "file"}
 }
 Function New-TargetPSTDirectory($MBX = $null) {
 <#
 .SYNOPSIS
 	Create a directory to hold the PST file while they are importing.
 	
 .DESCRIPTION
 	Create a directory to hold the PST file while they are importing.
 	These psts are deleted at the end, but reports for each PST file
 	are left in the directory.
 #>
 	$Return = "Error"
 
 	If ($MBX) {
 		$dir    = $Script:PSTShareDir + "\" + $MBX.samaccountname
 		if(!(Test-Path $dir)) {
 			$newdir = ni -ItemType Directory -Path $dir
 		} Else {
 			$Return = $dir
 		}
 	}
 	
 	If($newdir) {
 		$Return = (resolve-path $newdir).ProviderPath
 	}
 	$Return
 }
 Function Test-PSTIQLock () {
 <#
 .SYNOPSIS
 	Test to see if PST Import Queue is in use.
 .DESCRIPTION
 	Test to see if PST Import Queue is in use.
 #>
 	Test-Path $($Script:PSTUNCDir + '\PSTImportQueue.Lock')
 }
 Function Unlock-PSTIQ() {
 <#
 .SYNOPSIS
 	Remove the lock when the PST Import Queue is no longer in use.
 
 .DESCRIPTION
 	Remove the lock when the PST Import Queue is no longer in use.
 #>
 	if(Test-PSTIQLock) {Remove-Item $($Script:PSTUNCDir + '\PSTImportQueue.Lock')}
 }
 
 #=============================================================================
 #=============================================================================
 Function Send-NotificationInitialReport($JobObj) {
 <#
 .SYNOPSIS
 	Send the User a notification of PST files found and the work to be done.
 
 .DESCRIPTION
 	Send the User a notification of PST files found and the work to be done.
 	Incoming JobObj will give us the information we needed to the report.
 
 #>
 	$AllProcessFiles   = $JobObj | ?{$_.ProcessFile -eq $true}
 	$AllSkippedJobs	   = $JobObj | ?{$_.ProcessFile -ne $true}
 	$JobObj | %{$User = $_.UserMBX; $CLOK = $_.ClientVerOK }
 	
 	Write-Host $User
 	Write-Host $CLOK
 	$Subject = "Requested PST Import: Initial Report for " + $User
 	$From    = "EmailTeam@domain.com"
 	
 	$Body = "Note: This process will not delete or alter your original PSTs in any way.`n"
 	if ($JobObj.ClientVer -match "^14.") {
 		$Body	= $Body + "Your offline PST files will be copied into your new " + '"Archive - ' + $JogObj.UserEmail + '"' + " folder in Outlook.`n"
 	} Else {
 		$Body	= $Body + "Your offline PST files will be copied into your new " + '"Online PST"' + " folder in Outlook.`n"
 	}
 	$Body	= $Body + "`n"
 	$Body	= $Body + "The copy may take a few nights to complete. The process will start tonight about 7PM (1900). `n"
 	$Body	= $Body + "`t- Do not add any messages to your PSTs during this process.`n"
 	$Body	= $Body + "`t- Please shut down Outlook before leaving each night.`n"
 	$Body	= $Body + "`t- You do not need to turn off your PC.`n"
 	$Body	= $Body + "`n"
 	$Body	= $Body + "You will be notified when the copies are complete.`n"
 	$Body	= $Body + "`n"
 	#$Body	= $Body + "Click below to read the FAQ about Online PSTs."
 	#$Body	= $Body + "`n"
 	#$Body	= $Body + '"' + 'http://<link to self help doc on sharepoint>
 	#$Body	= $Body + "`n"
 	$Body	= $Body + "Once you see your new Online PST, use it as you would any PST.`n"
 	$Body	= $Body + "`n"
 	$Body	= $Body + "The procedure for finding your PST files has finished and found " + $JobObj.Count + " files. The ones scheduled for import, are:`n`n"
 
 	#"`n(http://office.microsoft.com/en-us/outlook-help/remove-a-password-from-a-personal-folders-file-pst-HA001151725.aspx)"
 	
 	if ($AllProcessFiles) {
 		$NUm = 0
 		$tmp = $null
 		$tmp = "{0,3}`t{1,-25}`t{2}`t`t{3}`n" -f ("Num"),("FileName"),("Size"),("Directory")
 		$Body	= $Body + $tmp
 		$AllProcessFiles | %{
 			$NUm++
 			$To		= $_.UserEmail
 			$temp	= '"' + (split-path $_.OrgUNCName) + '"'
 			$temp1	= $_.OrgFileName
 			$tempS	= $_.FileSize
 			$tmp = $null
 			$tmp = "{0,3}`t{1,-25}`t({2,7} M)`t{3}`n" -f ($Num),($temp1),($tempS), ($temp)
 			$Body	= $Body + $tmp	
 		}
 	} Else {
 		$Body	= $Body + "(None)"
 	}
 	$Body	= $Body + "`n`nSkipped Files: `n"
 	
 	if($AllSkippedJobs) {
 		$NUm = 0
 		$tmp = $null
 		$tmp = "{0,3}`t{1,6}`t{2,-25}`t{3}`t`t{4}`n" -f ("Num"),("Reason"),("FileName"),("Size"),("Directory")
 		$Body	= $Body + $tmp
 		$AllSkippedJobs | %{
 			$NUm++
 			$To		= $_.UserEmail
 			$temp	= '"' + (split-path $_.OrgUNCName) + '"'
 			$temp1	= $_.OrgFileName
 			$tempS	= $_.FileSize
 			$skip   = $_.SkipReason
 			$tmp = $null
 			$tmp = "{0,3}`t{1,6}`t{2,-25}`t({3,7} M)`t{4}`n" -f ($Num),($skip),($temp1),($tempS),($temp)
 			$Body	= $Body + $tmp
 		}
 	} Else {
 		$Body	= $Body + "(None)"
 	}
 	
 	$Body	= $Body + "`n`n"
 	$Body	= $Body + "Files are skipped due to either size (empty file) or not opened by Outlook in the last 2 years.`n"
 	$Body	= $Body + "Files also can be skipped if they look like Backups (BU) or Sharepoint Lists (SP).`n"
 	$Body	= $Body + "(Skipping files is just a suggestion. You can opt to include them.)`n"
 	
 		
 	if($JobObj.Count){
 		$Body	= $Body + "`n[Run on: " + $(Hostname) + "; Client is " + (($JobObj)[0].ClientVerOK) + " " + (($JobObj)[0].ClientVer) + "]["+(($JobObj)[0].IP)+"]["+(($JobObj)[0].OSName)+"]"
 		
 	} Else {
 		$Body	= $Body + "`n[Run on: " + $(Hostname) + "; Client is " + (($JobObj).ClientVerOK) + " " + (($JobObj).ClientVer) + "]["+(($JobObj).IP)+"]["+(($JobObj).OSName)+"]"
 		
 	}
 	If ($NoNotify) {
 		Send-MailMessage -Body $body -From $From -SmtpServer $Script:SMTPServer -Subject $Subject -To $Script:AdminEmail
 	} Else {
 		Send-MailMessage -Body $body -To $To -From $From -SmtpServer $Script:SMTPServer -Subject $Subject -cc $Script:AdminEmail
 	}
 	
 
 }
 Function Send-NotificationFinalReport($JobObj) {
 <#
 .SYNOPSIS
 	Send User the Final report detailing the work done.
 
 .DESCRIPTION
 	Send User the Final report detailing the work done.
 
 #>
 	$AllProcessFiles   = $JobObj | ?{$_.ProcessFile -eq "TRUE"}
 	$AllSkippedJobs	   = $JobObj | ?{$_.ProcessFile -eq "FALSE"}
 	
 	if($JobObj.Count) {
 		$Subject   = "Requested PST Import: Final Results for: " + $JobObj[0].UserMBX
 		$ClientVer = $JobObj[0].ClientVer
 		$Email     = $($JobObj[0].UserEmail.ToString())
 	} Else {
 		$Subject = "Requested PST Import: Final Results for: " + $JobObj.UserMBX
 		$ClientVer = $JobObj.ClientVer
 		$Email     = $($JobObj.UserEmail.ToString())
 	}
 	
 	$From    = "EmailTeam@domain.com"
 	
 	
 	$Body	= "The process for copying your PST files has finished.  Please take a few moments to complete the final steps:`n"
 	$Body	= $Body + "`n"
 	if ($ClientVer -match "^14." -or $ClientVer -match "Ignore") {
 		$Body	= $Body + "`t1. Review messages in the " + '"Archive - ' + $Email + '"' + " folder in Outlook and confirm all have been copied correctly.`n"
 	} Else {
 		$Body	= $Body + "`t1. Review messages in the " + '"Online PST"' + " folder in Outlook and confirm all have been copied correctly.`n"
 	}
 	#$Body	= $Body + "`t1. Review messages in the " + '"Online PST - <yourname>" or "Archive - <yourname>"' + " folder in Outlook and confirm all have been copied correctly.`n"
 	
 	
 	$Body	= $Body + "`t2. Disconnect the old PSTs from Outlook. (right click the name and choose: Close <pstname>)`n"
 	$Body	= $Body + "`t3. Restart Outlook - to release the lock on the file.`n"
 	$Body	= $Body + "`t4. Move the old PST files from your Home Drive (H:\) to a local drive. This will improve your Outlook performance!`n"
 	$Body	= $Body + "`t   (For example, create a directory in " + '"My Documents"' + " called " + '"Outlook Files"' + " and move the files listed below from your H:\ to the new Outlook Files directory.)`n"
 	
 	if ($ClientVer -match "^14." -or $ClientVer -match "Ignore") {
 		$Body	= $Body + "`t5. Begin storing messages in your " + '"Archive - ' + $Email + '"' + " folder instead of your old PSTs.`n"
 	} Else {
 		$Body	= $Body + "`t5. Begin storing messages in your " + '"Online PST"' + " folder instead of your old PSTs.`n"
 	}
 	#$Body	= $Body + "`n`tFor detailed instructions, click this link:`n"
 	#$Body	= $Body + "`n`t" + '"http://<sharepointsite>/Self-Help/Standard Application Issues/How do I Disconnect My Personal Folders.pdf"' + "`n"
 	#$Body	= $Body + "`n"
 	
 	$Body	= $Body + "`tNote: Remember you can read your old PSTs at any time. In a few days you will not be able to add any new emails to these old PST files.`n"
 	$Body	= $Body + "`t      You are now using the Online PST instead.`n"
 	$Body	= $Body + "`n"
 	$Body	= $Body + "`nThe files which can be moved off your home drive:`n"
 	$Body	= $Body + "`n"
 	$Body	= $Body + "`nIf you need assistance or have any questions, simply reply-all to this email.`n"
 	$Body	= $Body + "`n"
 	$Body	= $Body + "Imported Files:`n"
 	$NUm = 0
 	$tmp = "{0,3}`t{1,-40}`t{2,-35}`t{3}`t{4}`n" -f ("Num"),("FileName"),("Target Folder"),("Size"),("Directory")
 	$Body	= $Body + $tmp
 	$AllProcessFiles | %{
 		$NUm++
 		$To		=  $_.UserEmail
 		$temp	= '"' + (split-path $_.OrgUNCName) + '"'
 		$temp1	= $_.OrgFileName
 		$tempS	= $_.FileSize
 		$tempT  = $_.TargetRootFolder
 		$tmp = $null
 		$tmp = "{0,3}`t{1,-40}`t{2,-35}`t({3,7} M)`t{4}`n" -f ($Num),($temp1),($tempT),($tempS),($temp)
 		$Body	= $Body + $tmp	
 		
 	}
 	
 	$Body	= $Body + "`n`nSkipped Files:`n"
 	
 	if($AllSkippedJobs) {
 		$NUm = 0
 		$tmp = $null
 		$tmp = "{0,3}`t{1,6}`t{2,-40}`t{3}`t`t{4}`n" -f ("Num"),("Reason"),("FileName"),("Size"),("Directory")
 		$Body	= $Body + $tmp
 		$AllSkippedJobs | %{
 			$NUm++
 			$To		= $_.UserEmail
 			$temp	= '"' + (split-path $_.OrgUNCName) + '"'
 			$temp1	= $_.OrgFileName
 			$tempS	= $_.FileSize
 			$skip   = $_.SkipReason
 			$tmp = $null
 			$tmp = "{0,3}`t{1,6}`t{2,-40}`t({3,7} M)`t{4}`n" -f ($Num),($skip),($temp1),($tempS),($temp)
 			$Body	= $Body + $tmp
 		}
 	} Else {
 		$Body	= $Body + "(None) "
 	}
 	
 	#$Body	= $Body + "`n`nIf you have any questions, reply to this email."
 	$Body	= $Body + "`n"
 	$Body	= $Body + "`Thank You -- The Email Group"
 	$Body	= $Body + "`n"
 	
 	if($JobObj.Count){
 		$Body	= $Body + "`n[Run on: " + $(Hostname) + "; Client is " + (($JobObj)[0].ClientVerOK) + " " + (($JobObj)[0].ClientVer) + "]["+(($JobObj)[0].IP)+"]["+(($JobObj)[0].OSName)+"]"
 		
 	} Else {
 		$Body	= $Body + "`n[Run on: " + $(Hostname) + "; Client is " + (($JobObj).ClientVerOK) + " " + (($JobObj).ClientVer) + "]["+(($JobObj).IP)+"]["+(($JobObj).OSName)+"]"
 		
 	}
 	
 	Send-MailMessage -Body $body -To $To -From $From -SmtpServer $Script:SMTPServer -Subject $Subject -Cc $Script:AdminEmail
 	
 
 }
 Function Send-NotificationPSTRemoval() {
 <#
 .SYNOPSIS
 	Send user a report showing the files that are still 'connected' to Outlook
 
 .DESCRIPTION
 	Send user a report showing the files that are still 'connected' to Outlook
 	Connected means the LastWriteTime was within the last 24 hours.
 	
 	User gets this email every 14 days as a reminder to disconnect their Old
 	PST files and optionaly remove those PST files from Home Share.
 
 #>	
 	#------------------------------------------------------------------------
 	
 	Param(
 		$ArchObj, 
 		[array]$PSTFilesInUse,
 		[array]$PSTFilesALL,
 		[array]$PSTFilesMovable
 	)
 	
 	
 	$Subject = "Online PST -- Possible Outlook Issue : " + $ArchObj.DisplayName + " (" + $ArchObj.DaysSinceImport + ")"
 	$From    = "EmailTeam@domain.com"
 	$To 	 = $((get-mailbox $ArchObj.DisplayName).PrimarySMTPAddress.ToString())
 	$CC      = $Script:AdminEmail
 	
 	[int]$InUse   = cc $PSTFilesInUse; 		Write-Verbose "[SendNotification] Inuse  - $InUse"
 	[int]$AllPSTs = cc $PSTFilesALL; 		Write-Verbose "[SendNotification] AllPST - $AllPSTs"
 	[int]$Movable = cc $PSTFilesMovable;	Write-Verbose "[SendNotification] AllPST - $AllPSTs"
 	
 	$Vers = (import-csv $Script:PSTCompleteFile | ?{$_.UserMBX -match ($ArchObj.DisplayName)} | sort LastCheckTime -Descending)
 	
 	If ($Vers.Count) {
 		$ClientVer = $Vers[0].ClientVer
 	} Else {
 		If ($Vers){
 			$ClientVer = $Vers.ClientVer
 		} Else {
 			$ClientVer = "Ignore"
 		}
 	}
 	Write-Verbose "[SendNotification] $(($ArchObj).DisplayName) : Client Ver is $($ClientVer)"
 	
 	If ($ClientVer -match "^14." -or $ClientVer -match "Ignore") {
 		$Archive =
`

