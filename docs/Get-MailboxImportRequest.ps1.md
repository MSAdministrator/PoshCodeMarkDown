---
Author: jon webster
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: utf-8
License: cc0
PoshCode ID: 2792
Published Date: 2014-07-14t21
Archived Date: 2014-02-27t20
---

# get-mailboximportrequest - 

## Description

this exchange 2010 mailboximportrequest cmdlet is to help identify imports that may never complete successfully or are taking a long time to complete so they can be suspended and other imports in the queued can complete in a timely manner. this version fixes pipeline input support and improves type handling.

## Comments



## Usage



## TODO



## script

`set-defaultproperties`

## Code

`#
 #
 #
 #
 #
 [CmdletBinding()]
 PARAM (
 	[parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]$Identity
 )
 
 BEGIN {
 	Function Set-DefaultProperties {
 		PARAM (
 			[string]$name,
 			[string[]]$DefaultProperties
 		)
 
 		$xml = "<?xml version='1.0' encoding='utf-8' ?><Types><Type>"
 		$xml += "<Name>$($name)</Name>"
 		$xml += "<Members><MemberSet><Name>PSStandardMembers</Name><Members>"
 		$xml += "<PropertySet><Name>DefaultDisplayPropertySet</Name><ReferencedProperties>"
 
 		foreach( $default in $DefaultProperties ) {
 			$xml += "<Name>$($default)</Name>"
 		}
 
 		$xml += "</ReferencedProperties></PropertySet></Members></MemberSet></Members>"
 		$xml += "</Type></Types>"
 
 		$file = "$($env:Temp)\$name.ps1xml"
 
 		Out-File -FilePath $file -Encoding "UTF8" -InputObject $xml -Force
 
 		$typeLoaded = $host.Runspace.RunspaceConfiguration.Types | where { $_.FileName -eq  $file }
 
 		if( $typeLoaded -ne $null ) {
 			Write-Verbose "Type Loaded"
 			Update-TypeData
 		}
 		else {
 			Update-TypeData $file
 		}
 	}
 	
 	Set-DefaultProperties -Name $customObjectName @(�TargetAlias','StatusDetail','BytesTransferred','ItemsTransferred','ItemsLeft','Item %','Total %','Durration')
 }
 
 PROCESS
 {
 	if($_)
 	{
 		if($_.Identity.GetType() -eq [Microsoft.Exchange.MailboxReplicationService.RequestJobObjectId] -or $_.Identity.GetType() -eq [Microsoft.Exchange.MailboxReplicationService.RequestIndexEntryObjectId])
 		{
 			$temp = $_|Get-MailboxImportRequestStatistics|select @{n="ItemsLeft";e={$_.estimatedtransferitemcount - $_.itemstransferred}},@{n="Item %";e={[int]($_.itemstransferred/$_.estimatedtransferitemcount * 100)}},@{n="Total %";e={$_.percentcomplete}},@{n="Durration";e={$_.TotalInProgressDuration}},*
 		} else { Write-Warning "Invalid Mailbox Import Request ID"; return }
 
 	} else {
 		$temp = Get-MailboxImportRequest -status InProgress|Get-MailboxImportRequestStatistics|select @{n="ItemsLeft";e={$_.estimatedtransferitemcount - $_.itemstransferred}},@{n="Item %";e={[int]($_.itemstransferred/$_.estimatedtransferitemcount * 100)}},@{n="Total %";e={$_.percentcomplete}},@{n="Durration";e={$_.TotalInProgressDuration}},*
 	}
 
 	$temp |% {
 		$_.PSObject.TypeNames.Insert(0,$customObjectName)
 	}
 	$temp
 }
`

