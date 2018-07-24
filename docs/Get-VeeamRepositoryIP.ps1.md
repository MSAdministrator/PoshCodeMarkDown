---
Author: robin olsson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5707
Published Date: 2015-01-24t00
Archived Date: 2015-01-25t23
---

# get-veeamrepositoryip - 

## Description

used to find veeam error events where backup failed because of rpc errors [dorpcwithbinary]. outputs an array of the unique ip adresses it finds in the eventlog and finally appends the queried veeamserver to the list

## Comments



## Usage



## TODO



## script

`get-veeamrepositoryip`

## Code

`#
 #
 <#
 	.SYNOPSIS
 		Finds the unique IP(s) of the Veeam repository server(s) that failed to backup one or more virtual servers because of RPC errors.
 	
 	.DESCRIPTION
 		Extracts the unique IP adresses for the Veeam repository servers that report RPC errors in the eventlog
         so it can be used to report an error or pass IP along to a script that restarts the offending machines. 
 
 	.PARAMETER ComputerName
 		The computername for the Veeam backupserver that holds the eventlog information.
 	
 	.PARAMETER StartTime
 		Decides how many hours to go back in the eventlog. Default is 24 hours.
 	
 	.EXAMPLE
 				PS C:\> Get-VeeamRepositoryIP -ComputerName vmbackup01 -StartTime 24
 	
 	.NOTES
 		If one or more matches are found then the Veeam backupserver specified in the ComputerName variable is added to the end of the list.
 #>
 function Get-VeeamRepositoryIP
 {
 	[CmdletBinding()]
 	[OutputType([array])]
 	param
 	(
 		[Parameter(Mandatory = $true,
 				   ValueFromPipeline = $true,
 				   ValueFromPipelineByPropertyName = $true,
 				   Position = 1)]
 		[ValidateNotNullOrEmpty()]
 		[string]
 		$ComputerName,
 		[Parameter(Mandatory = $false,
 				   Position = 2)]
 		[int32]
 		$StartTime = 24
 	)
 	
 	Begin
 	{
 		$STime = (Get-Date).AddHours(-$StartTime)
 		$Regex = �\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b�
 	}
 	Process
 	{
 		try
 		{
 			[array]$IP = Get-WinEvent -ComputerName $ComputerName -FilterHashtable @{Logname = "Veeam Backup"; ProviderName = "Veeam MP"; ID = "190"; Level = "2"; StartTime = $STime} -ErrorAction Stop | Where-Object { $_.message -like '*`[DoRpcWithBinary]*' } | ForEach-Object { $_.message | Select-String -Pattern $Regex -AllMatches | % { $_.Matches } | % { $_.Value } } | Sort-Object -Unique
 		}
 		catch
 		{
 			Write-Error "No event found"
 		}
         if ($IP -ne $Null)
         {
             $IP += $ComputerName
         }
 	}
 	End
 	{
 		return $IP
 	}
 }
`

