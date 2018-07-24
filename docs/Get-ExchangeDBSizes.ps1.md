---
Author: karl mitschke
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 3694
Published Date: 2014-10-16t19
Archived Date: 2014-09-08t01
---

# get-exchangedbsizes - 

## Description

gathers data from exchange mailbox servers.

## Comments



## Usage



## TODO



## script

`get-mountpointinfo`

## Code

`#
 #
 <#
 .SYNOPSIS
         Get-ExchangeDBSizes - Gather data on Exchange 2007 / 2010 Mailbox Databases.
 .DESCRIPTION
         Gathers data from Exchange mailbox servers.
 		These data include:
 		Server\StorageGroup\Database (2007) or Database (2010),
 		Total Size (in GB) of the disk,
 		Size of the .edb file (in GB),
 		Free disk space,
 		Percent Disk Free,
 		Total Mailboxes on the database,
 		White Space,
 		Total Free disk space,
 		Total Percent Free disk space
 .EXAMPLE
         Get-ExchangeDBSizes 
 .LINK
 		http://unlockpowershell.wordpress.com/
 		http://poshcode.org/1901
 #>
 
  
 Function Get-MountPointInfo($ServerName) { 
 $Summary = @() 
 $VolumeHash = @{}
 $VolumeCapacityHash = @{}
 $DatabaseSizeHash = @{}
 $DatabaseFreeHash = @{}
 $MountPoints = Get-WmiObject -Class Win32_Mountpoint -ComputerName $ServerName
 $MailboxDatabases = Get-MailboxDatabase -Server $ServerName
 $Volumes = Get-WmiObject -Class Win32_Volume -ComputerName $ServerName | Where-Object {$_.DriveType -eq 3}| Select-Object Name,FreeSpace,Capacity 
 $DatabaseLetters = $MailboxDatabases | Select-Object edbfilepath
 foreach ($Volume in $Volumes) {
 $VolumeHash.Add($Volume.name.TrimEnd("\"),$Volume.FreeSpace)
 $VolumeCapacityHash.Add($Volume.name.TrimEnd("\"),$Volume.Capacity)
 }
 foreach ($drive in $DatabaseLetters)
 {
 	$letter = $drive.EdbFilePath.Substring(0,$drive.EdbFilePath.lastIndexOf("\"))
 	$DatabaseSizeHash.Add($letter.TrimEnd("\"),$VolumeCapacityHash[$letter])
 	$DatabaseFreeHash.Add($letter.TrimEnd("\"),$VolumeHash[$letter])
 }
 $VolumeHash.Clear()
 $VolumeCapacityHash.Clear()
 foreach ($MP in $Mountpoints) { 
 $MP.directory = $MP.directory.replace("\\","\").Split("=")[1].Replace("`"","") 
 if($DatabaseSizeHash[$MP.directory])
 {
 $Record = new-Object PSObject
 $OSfree = $DatabaseFreeHash[$MP.directory]
 $OSCapacity = $DatabaseSizeHash[$MP.directory]
 $DestFolder = "\\"+$ServerName + "\" + $MP.directory
 $colItems = (Get-ChildItem $destfolder.Replace(":","$") -Recurse| where{$_.length -ne $null} |Measure-Object -property length -sum) 
 if($colItems.sum -eq $null) { 
 $fsize = 0 
 } else { 
 $fsize = $colItems.sum 
 } 
 $TotFolderSize = $fsize + $OSfree 
 $percFree = "{0:P0}" -f ( $OSfree/$TotFolderSize) 
 $Record | add-Member -memberType noteProperty -name Server -Value $ServerName 
 $Record | add-Member -memberType noteProperty -name "Mount Point" -Value $MP.directory
 $Record | add-Member -memberType noteProperty -name "Capacity" -Value ("{0:N2}" -f ($OSCapacity /1gb))
 $Record | add-Member -memberType noteProperty -name "Used" -Value ("{0:N2}" -f ($fsize / 1gb)) 
 $Record | add-Member -memberType noteProperty -name "Free" -Value ("{0:N2}" -f ($OSfree / 1gb)) 
 $Record | add-Member -memberType noteProperty -name "Percent Free" -Value $percFree 
 $Summary += $Record 
 } 
 } 
 return $Summary 
 } 
 function Get-ExchangeWhiteSpace { 
 param( 
    $ComputerName = $(throw "ComputerName cannot be empty.") 
 )
 
 $tc = [System.Management.ManagementDateTimeconverter] 
 $Start =$tc::ToDmtfDateTime( (Get-Date).AddDays(-1).Date ) 
 $End =$tc::ToDmtfDateTime( (Get-Date).Date) 
 
 $whiteSpace = @{}
 
 $DB = @{Name="DB";Expression={$_.InsertionStrings[1]}} 
 $FreeMB = @{Name="FreeMB";Expression={[int]$_.InsertionStrings[0]}} 
 
 $freespace = Get-WMIObject Win32_NTLogEvent -ComputerName $ComputerName -Filter "LogFile='Application' AND EventCode=1221 AND TimeWritten>='$Start' AND TimeWritten<='$End'" | Select-Object $DB,$FreeMB | Sort-Object DB �Unique �Descending
 $freespace | ForEach-Object {$whiteSpace.Add($_.DB,$_.FreeMB)}
 }
 $AllServers = @()
 $ServerVersion = @{}
 foreach ($server in Get-MailboxServer)
 {
 	$ServerName = $server.name
 	$version = Get-ExchangeServer -Identity $ServerName | Select-Object AdminDisplayVersion
 	if($version.admindisplayversion.major)
 	{
 		$ServerVersion.Add($ServerName,$version.admindisplayversion.major)
 	}
 	else
 	{
 		$ServerVersion.Add($ServerName,$version.admindisplayversion.Split()[1].split(".")[0])
 	}	
 }
 foreach ($server in Get-MailboxServer)
 {
 	$ServerName = $server.Name
 	if ([int]$ServerVersion[$ServerName] -gt 8)
 		$whiteSpace = @{}
 		$Free = Get-MailboxDatabase -Server $ServerName -Status | Select-Object Server,Name,AvailableNewMailboxSpace
 		foreach ($item in $free)
 		{
 			$db = $ServerName+"\"+$item.Name
 			$whiteSpace.Add($item.Name,[int]$item.AvailableNewMailboxSpace.split("(")[1].Split()[0]/1mb)
 		}
 	}
 	Else
 		. Get-ExchangeWhiteSpace $ServerName
 	}
 	$MountPoint = . Get-MountPointInfo $ServerName
 	foreach ($objItem in Get-MailboxDatabase -Server $ServerName)
     {
     $edbfilepath = $objItem.edbfilepath
      $path = "`\`\" + $ServerName + "`\" + $objItem.EdbFilePath.Split(":")[0] + "$"+ $objItem.EdbFilePath.Split(":")[1]
     $dbsize = Get-ChildItem $path
 	$dbpath=(Split-Path $edbfilepath.Split(":")[1] -Leaf).trimend(".edb")
 	if (!$MountPoint)
 	{
 		$mailboxpath = $ServerName + $edbfilepath.Split(":")[1].trimend(".edb")
 		$size = get-wmiobject -computername $ServerName win32_logicaldisk |where-object {$_.deviceid -eq $objItem.EdbFilePath.Split("\")[0]} |select-object deviceID, Freespace, Size
 		$freespace = ($size.freespace / 1GB)
 		$total = ($size.size / 1GB)
 	}
 	if ($MountPoint)
 	{
 		$mailboxpath = "$ServerName\$dbpath"
 		$MPPath = $EdbFilePath.Substring(0,$EdbFilePath.LastIndexOf("\"))
 		$freespace = $DatabaseFreeHash[$MPPath ] /1GB
 		$total = $DatabaseSizeHash[$MPPath] /1GB
 	}
 	$PercentFree = "{0:n2}" -f ($freespace / $total *100)
 	if (!$MountPoint)
 	{
 		$mailboxcount = Get-MailboxStatistics -database "$mailboxpath"  |Where {$_.DisconnectDate -eq $null -and $_.ObjectClass -eq 'Mailbox'} |measure-object
 	}
 	if($MountPoint)
 	{
 		$mailboxcount = Get-MailboxStatistics -database $mailboxpath.Split("\")[1]  |Where {$_.DisconnectDate -eq $null -and $_.ObjectClass -eq 'Mailbox'} |measure-object
 	}
 	
 	$ReturnedObj = New-Object PSObject
 	$ReturnedObj | Add-Member -MemberType NoteProperty -Name "Server\StorageGroup\Database" -Value $objItem.Identity
 	$ReturnedObj | Add-Member -MemberType NoteProperty -Name "Total Size (GB)" -Value ("{0:n2}" -f ($total))
 	$ReturnedObj | Add-Member -MemberType NoteProperty -Name "Used Space (GB)" -Value ("{0:n2}" -f ($dbsize.Length/1024MB))
 	$ReturnedObj | Add-Member -MemberType NoteProperty -Name "Free Space (GB)" -Value ("{0:n2}" -f ($freespace))
 	$ReturnedObj | Add-Member -MemberType NoteProperty -Name "Percent Disk Free" -Value $percentfree
 	$ReturnedObj | Add-Member -MemberType NoteProperty -Name "User Mailbox Count" -Value $mailboxcount.count
 	if ($objitem.Identity.Split("\").Count -eq 3){$dbasename = $objitem.Identity.Substring($objitem.Identity.IndexOf("\")+1)}
 	else{$dbasename = $objitem.Identity}
 	$ReturnedObj | Add-Member -MemberType NoteProperty -Name "White Space (GB)" -Value ("{0:n2}" -f ($whiteSpace[$dbasename]/1024))
 	$ReturnedObj | Add-Member -MemberType NoteProperty -Name "Total Free (GB)" -Value ("{0:n2}" -f ($freespace + $whiteSpace[$dbasename]/1024))
 	$TotalPercent = ($freespace + $whiteSpace[$dbasename]/1024) / $total *100
 	$ReturnedObj | Add-Member -MemberType NoteProperty -Name "Total Percent Free" -Value ("{0:n2}" -f ($TotalPercent))
     $AllServers += $ReturnedObj
 	}
 }
 $AllServers |Export-Csv C:\scripts\MailboxDatabaseSizeReport.csv -NoTypeInformation
`

