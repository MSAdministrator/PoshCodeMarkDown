---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3780
Published Date: 2012-11-23t10
Archived Date: 2012-11-30t08
---

# get- exchangembstore - 

## Description

get mailbox store info per exchange 2003 server

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 Param (
 [Parameter(Position=0,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$true)]
 [alias("Name","ComputerName")]$Computer=@("xcserver01")
 )
 
 process{
 $XCinfo = GetXCDatabases $Computer
 $XCMaintInfo = GetMBStoreMaintenance $Computer
 foreach ($DB in $XCinfo){
 	try{
 		$DB.WhiteSpace = $XCMaintInfo[$DB.MB].WhiteSpace
 		$DB.RetainedMailbox = $XCMaintInfo[$DB.MB].RetainedMailbox
 		$DB.RetainedMailboxSize = $XCMaintInfo[$DB.MB].RetainedMailboxSize
 		$DB.DeletedMailbox = $XCMaintInfo[$DB.MB].DeletedMailbox
 		$DB.DeletedMailboxSize = $XCMaintInfo[$DB.MB].DeletedMailboxSize
 		$DB.DeletedItems = $XCMaintInfo[$DB.MB].DeletedItems
 		$DB.DeletedItemsSize = $XCMaintInfo[$DB.MB].DeletedItemsSize
 		}
 	catch{
 		Write-Error $("For " + $Computer + ": " + $($error[0]))
 		}
 	}
 $Global:XCserverInfo += $XCinfo | select Server, StorageGroup, DatabaseName, Path, FileSize, DeletedMailbox, DeletedMailboxSize, RetainedMailbox, RetainedMailboxSize, DeletedItems, DeletedItemsSize 
 }
 
 begin{
 $Global:XCserverInfo = @()
 $Quote = [char]34
 
 Function GetADSIObject ($Name){
 write-verbose "get ADSI information for $Name"
 $root = [ADSI]'LDAP://RootDSE'
 $cfConfigRootpath = "LDAP://" + $root.ConfigurationNamingContext.tostring()
 $configRoot = [ADSI]$cfConfigRootpath
 $searcher = new-object System.DirectoryServices.DirectorySearcher($configRoot)
 $searcher.Filter = '(&(objectCategory=msExchExchangeServer)(cn=' + $Name + '))'
 $searchres = $searcher.FindOne()
 $snServerEntry = New-Object System.DirectoryServices.directoryentry
 $snServerEntry = $searchres.GetDirectoryEntry()
 $adsiServer = [ADSI]('LDAP://' + $snServerEntry.DistinguishedName)
 return $adsiserver
 }
 
 Function GetXCDatabases($XCservername){
 $XCDatabases = @()
 $XCserver = GetADSIObject $XCservername
 write-verbose "get XC DB information for $($XCserver.Name)"
 $dfsearcher = new-object System.DirectoryServices.DirectorySearcher($XCserver)
 $dfsearcher.Filter = "(objectCategory=msExchPrivateMDB)"
 $srSearchResults = $dfsearcher.FindAll()
 $dfsearcher.Filter = "(objectCategory=msExchPublicMDB)"
 $srSearchResults += $dfsearcher.FindAll()
 foreach ($srSearchResult in $srSearchResults){
 	$msMailStore = $srSearchResult.GetDirectoryEntry()
 	$msMailStoreFileName = $msMailStore.msExchEDBFile
 	$Filter = $msMailStoreFileName -replace '\\','\\'
 	write-verbose "getting details for $Filter thru WMI"
 	$msMailStoreFile = (Gwmi -computername $XCservername -class CIM_DataFile -filter "Name = '$Filter'")
 	$sgStorageGroup = $msMailStore.psbase.Parent
 	if ($sgStorageGroup.msExchESEParamBaseName -ne "R00"){
 		$XCdatabase = New-Object PSObject -Property @{
 			Server = [string] $XCservername
 			StorageGroup = [string] $sgStorageGroup.Name
 			DataBaseName = [string] $msMailStore.Name
 			MB = "$($sgStorageGroup.Name)\$($msMailStore.Name)"
 			Path = [string] $msMailStoreFileName
 			FileSize = "$([math]::round($msMailStoreFile.FileSize/1048576,0)) MB"
 			WhiteSpace = ""
 			RetainedMailbox = ""
 			RetainedMailboxSize = ""
 			DeletedMailbox = ""
 			DeletedMailboxSize = ""
 			DeletedItems = ""
 			DeletedItemsSize = ""
 			}
 		}
 	write-verbose "retrieved information for $($XCdatabase.MB)"
 	$XCDatabases += $XCdatabase
 	}
 return $XCdatabases | sort MB
 }
 
 Function GetMBStoreMaintenance ($Computername){
 $MBMaintInfo = @{}
 $WmidtQueryDT = [System.Management.ManagementDateTimeConverter]::ToDmtfDateTime([DateTime]::UtcNow.AddDays(-1))
 $Filter = "Logfile='Application' and SourceName = 'MSExchangeIS Mailbox Store' and TimeWritten >='" + $WmidtQueryDT + "'"
 $Events = (Get-WmiObject -Computername $ComputerName -class Win32_NTLogEvent -Filter $Filter | sort $_.TimeWritten -Descending)
 foreach ($Event in $Events){
 	write-verbose "getting details of Event $($Event.Eventcode)"
 	switch ($Event.Eventcode){
 			$itemsremovedStart = $Event.Insertionstrings[0]
 			$itemsremovedSizeStart = $Event.Insertionstrings[1]
 			$itemsremovedEnd = $Event.Insertionstrings[2]
 			$itemsremovedSizeEnd = $Event.Insertionstrings[3]
 			$MB = $Event.Insertionstrings[4]
 			write-verbose "For: $MB Item Cleanup details"
 			write-verbose "Begin: $itemsremovedStart items with a total size of $itemsremovedStartSize KB"
 			write-verbose "End: $itemsremovedEnd items with a total size of $itemsremovedEndSize KB"
 			if (-not $MBMaintInfo.ContainsKey($MB)){
 				$MBMaintInfo.add($MB,@{})
 				$MBMaintInfo.$MB.Name = $MB
 				}
 			if (-not $MBMaintInfo.$MB.ContainsKey("DeletedItems")){$MBMaintInfo.$MB.Add("DeletedItems",$itemsremovedEnd)}
 			if (-not $MBMaintInfo.$MB.ContainsKey("DeletedItemsSize")){$MBMaintInfo.$MB.Add("DeletedItemsSize","$itemsremovedSizeEnd KB")}
 			}
 			$WhiteSpace = $Event.Insertionstrings[0]
 			$MB = $Event.Insertionstrings[1]
 			write-verbose "For: $MB Online Defrag details"
 			write-verbose "Defrag has freed up $WhiteSpace MB"
 			if (-not $MBMaintInfo.ContainsKey($MB)){
 				$MBMaintInfo.add($MB,@{})
 				$MBMaintInfo.$MB.Name = $MB
 				}
 			if (-not $MBMaintInfo.$MB.ContainsKey("WhiteSpace")){$MBMaintInfo.$MB.Add("WhiteSpace","$WhiteSpace MB")}
 			}
 			$mbremoved = $Event.Insertionstrings[0]
 			$mbremovedSize = $Event.Insertionstrings[1]
 			$mbretained = $Event.Insertionstrings[2]
 			$mbretainedSize = $Event.Insertionstrings[3]
 			$MB = $Event.Insertionstrings[4]
 			write-verbose "For: $MB Mailbox Cleanup details"
 			write-verbose "Removed: $mbremoved mailboxes with a total size of $mbremovedSize KB"
 			write-verbose "Retained: $mbretained mailboxes with a total size of $mbretainedSize KB"
 			if (-not $MBMaintInfo.ContainsKey($MB)){
 				$MBMaintInfo.add($MB,@{})
 				$MBMaintInfo.$MB.Name = $MB
 				}
 			if (-not $MBMaintInfo.$MB.ContainsKey("DeletedMailbox")){$MBMaintInfo.$MB.Add("DeletedMailbox",$mbremoved)}
 			if (-not $MBMaintInfo.$MB.ContainsKey("DeletedMailboxSize")){$MBMaintInfo.$MB.Add("DeletedMailboxSize","$mbremovedSize KB")}
 			if (-not $MBMaintInfo.$MB.ContainsKey("RetainedMailbox")){$MBMaintInfo.$MB.Add("RetainedMailbox",$mbretained)}
 			if (-not $MBMaintInfo.$MB.ContainsKey("RetainedMailboxSize")){$MBMaintInfo.$MB.Add("RetainedMailboxSize","$mbretainedSize KB")}
 			}
 		'default'{write-verbose "event $($Event.Eventcode) not defined"}
 		}
 		}
 	return $MBMaintInfo | Sort Name
 }
 
 }
 
 
 end{
 $Global:XCserverInfo | sort Server, StorageGroup, DatabaseName
 }
`

