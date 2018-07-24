---
Author: lazywinadmin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3937
Published Date: 2015-02-10t20
Archived Date: 2015-05-12t00
---

# get-disksizeinfo - 

## Description

winter scripting games 2013 – practice event #1

## Comments



## Usage



## TODO



## function

`get-disksizeinfo`

## Code

`#
 #
 Function Get-DiskSizeInfo {
         <#
         .DESCRIPTION
         Check the Disk(s) Size and remaining freespace.
 
         .PARAMETER ComputerName
         Specify the computername(s)
 
         .INPUTS
         System.String
 
         .OUTPUTS
         System.Management.Automation.PSObject
 
         .EXAMPLE
         Get-DiskSizeInfo
         
         Get the drive(s), Disk(s) space, and the FreeSpace (GB and Percentage)
 
         .EXAMPLE
         Get-DiskSizeInfo -ComputerName SERVER01,SERVER02
 
         Get the drive(s), Disk(s) space, and the FreeSpace (GB and Percentage)
 	on the Computers SERVER01 and SERVER02
 
 		.EXAMPLE
         Get-Content Computers.txt | Get-DiskSizeInfo
 
         Get the drive(s), Disk(s) space, and the FreeSpace (GB and Percentage)
 	for each computers listed in Computers.txt
 
         .NOTES
     	NAME  : Get-DiskSizeInfo
 	AUTHOR: Francois-Xavier Cat
 	EMAIL : fxcat@LazyWinAdmin.com
 	DATE  : 2013/02/05 
 	
 	.LINK
 	http://lazywinadmin.com
 
         #>
 
 	[CmdletBinding()]
 	param(
 		[Parameter(ValueFromPipeline=$True)]
 		[string[]]$ComputerName = $env:COMPUTERNAME
 	)
 		}
 	PROCESS {
 		Foreach ($Computer in $ComputerName) {
 			
 			Write-Verbose -Message "ComputerName: $Computer - Getting Disk(s) information..."
 			try {
 				$params = @{'ComputerName'=$Computer;
 	            			'Class'='Win32_LogicalDisk';
 							'Filter'="DriveType=3";
 							'ErrorAction'='SilentlyContinue'}
 				$TryIsOK = $True
 				
 				$Disks = Get-WmiObject @params
 			
 			Catch {
             	"$Computer" | Out-File -Append -FilePath c:\Errors.txt
             	$TryIsOK = $False
 			
 			if ($TryIsOK) {
 				Write-Verbose -Message "ComputerName: $Computer - Formating information for each disk(s)"
             	foreach ($disk in $Disks) {
 					
 					Write-Verbose -Message "ComputerName: $Computer - $($Disk.deviceid)"
 					$output =	 	@{'ComputerName'=$computer;
                     				'Drive'=$disk.deviceid;
 									'FreeSpace(GB)'=("{0:N2}" -f($disk.freespace/1GB));
 									'Size(GB)'=("{0:N2}" -f($disk.size/1GB));
 									'PercentFree'=("{0:P2}" -f(($disk.Freespace/1GB) / ($disk.Size/1GB)))}
 					
 					$object = New-Object -TypeName PSObject -Property $output
                 	$object.PSObject.TypeNames.Insert(0,'Report.DiskSizeInfo')
 					
 					Write-Output -InputObject $object
 					
 				
 			
 		
 		}
`

