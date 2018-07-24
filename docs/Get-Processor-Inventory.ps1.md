---
Author: kevin kirkpatrick
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5116
Published Date: 2014-04-23t15
Archived Date: 2014-04-26t17
---

# get processor inventory - 

## Description

script originally developed for internal dbas to get information on processor core counts, which is important for licensing. this script includes some basic error handling and full comment based help. output properties are computer, socket (designation), core count, logical processors, hyperthreading enabled, description and type (physical/virtual).

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 	.SYNOPSIS
 		Returns information about the Processor via WMI. 
 
 	.DESCRIPTION
 		Information about the processor is returned using the Win32_Processor WMI class. 
 
 		You can provide a single computer/server name or supply an array/list.
 
 		Operating systems prior to Windows 7 & Server 2008 may not return accurate results due to the difference in WMI versions.
 
 	.PARAMETER  Computers
 		Single computer name, list of computers or .txt file containing a list of computers.
 
 	.EXAMPLE
 		.\Get-ProcessorInventory.ps1 -Computers (Get-Content C:\ComputerList.txt)
 
 	.EXAMPLE
 		.\Get-ProcessorInventory.ps1 -Computers Test-Server.company.com
 
 	.EXAMPLE
 		.\Get-ProcessorInventory.ps1 -Computers SERVER1.company.com,SERVER2.company.com | Format-Table -AutoSize
 
 	.EXAMPLE
 		.\Get-ProcessorInventory.ps1 -Computers SERVER1.company.com,SERVER2.company.com | Export-Csv C:\ProcInv.csv -NoTypeInformation
 
 	.INPUTS
 		System.String
 
 	.OUTPUTS
 		Selected.System.Management.ManagementObject
 
 	.NOTES
 		#=======================================================
 		Author: Kevin Kirkpatrick
 		Created: 4/16/14
 
 		Disclaimer: This script comes with no implied warranty or guarantee and is to be used at your own risk. It's recommended that you TEST
 		execution of the script against Dev/Test before running against any Production system. 
 
 		#========================================================
 
 	.LINK 
 		https://github.com/vN3rd/PowerShell-Scripts
 
 	.LINK
 		about_WMi
 
 	.LINK
 		about_Wmi_Cmdlets
 #>
 
 
 [cmdletbinding()]
 Param (
 	[parameter(Mandatory = $true,
 			   ValueFromPipeline = $true,
 			   HelpMessage = "Enter the name of a computer or an array of computer names")]
 	[system.string[]]$Computers
 )
 
 $ErrorActionPreference = "Stop"
 
 foreach ($C in $Computers)
 {
 	$i++
 	
 	if (Test-Connection -ComputerName $C -Count 1 -Quiet)
 	{
 		try
 		{
 			[string]$OSCheck = (Get-WmiObject -Query "SELECT Caption FROM win32_operatingsystem" -ComputerName $C).caption
 			
 			if (($OSCheck -like "*7*") -or ($OSCheck -like "*8*") -or ($OSCheck -like "*12*")) { }
 			else { Write-Warning -Message "$C is running '$OSCheck', which may return inaccurate results" }
 			
 			#================================
 			
 			$Type = @{
 				label = 'Type'
 				expression = {
 					if ($_.L2CacheSize -eq '0') { "Virtual" }
 					else { "Physical" }
 				}
 			
 			$HyperThreading = @{
 				label = 'HyperthreadingEnabled'
 				expression = {
 					if ($_.NumberOfLogicalProcessors -gt $_.NumberOfCores) { "Yes" }
 					elseif ($_.NumberOfLogicalProcessors -eq $null) { "Required WMI Properties Not Available in OS"}
 					else { "No" }
 				}
 			
 			$ComputerName = @{ label = 'Computer'; Expression = { $_.PSComputerName } }
 			$CoreCount = @{ label = 'CoreCount'; Expression = { $_.NumberOfCores } }
 			$LogicalCores = @{ label = 'LogicalProcessors'; expression = { $_.NumberOfLogicalProcessors } }
 			$Description = @{ label = 'Description'; Expression = { $_.Name } }
 			$Socket = @{ label = 'Socket'; expression = { $_.SocketDesignation } }
 			#================================
 			
 			Get-WmiObject -Query "SELECT * FROM win32_processor" -ComputerName $C |
 			Select-Object $ComputerName, $Socket, $CoreCount, $LogicalCores, $HyperThreading, $Description, $Type
 			
 			
 			
 		
 		catch
 		{
 			Write-Warning "$C - $_"
 		
 	
 	else
 	{
 		Write-Warning "$C is unreachable"
 		
 	
 	$TotalComputers = $Computers.Length
 	$PercentComplete = [int](($i / $TotalComputers) * 100)
 	Write-Progress -Activity "Working on $C..." -CurrentOperation "$PercentComplete% Complete" -Status "Percent Complete" -PercentComplete $PercentComplete
 	
`

