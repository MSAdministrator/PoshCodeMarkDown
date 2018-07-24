---
Author: francois-xavier cat
Publisher: 
Copyright: 
Email: fxcat@lazywinadmin.com
Version: 2013.06.03
Encoding: ascii
License: cc0
PoshCode ID: 4191
Published Date: 2013-06-04t03
Archived Date: 2013-06-11t08
---

# get-localgroupmembership - function-get-localgroupmembership.ps1

## Description

this function get the local group membership on a local or remote computer using adsi/winnt. by default the function will run on the localhost ($env

## Comments



## Usage



## TODO



## function

`get-localgroupmembership`

## Code

`#
 #
 #
 
 Function Get-LocalGroupMembership {
 <#
 .Synopsis
     Get the local group membership.
             
 .Description
     Get the local group membership.
             
 .Parameter ComputerName
     Name of the Computer to get group members. Default is "localhost".
             
 .Parameter GroupName
     Name of the GroupName to get members from. Default is "Administrators".
             
 .Example
     Get-LocalGroupMembership
     Description
     -----------
     Get the Administrators group membership for the localhost
             
 .Example
     Get-LocalGroupMembership -ComputerName SERVER01 -GroupName "Remote Desktop Users"
     Description
     -----------
     Get the membership for the the group "Remote Desktop Users" on the computer SERVER01
 
 .Example
     Get-LocalGroupMembership -ComputerName SERVER01,SERVER02 -GroupName "Administrators"
     Description
     -----------
     Get the membership for the the group "Administrators" on the computers SERVER01 and SERVER02
 
 .OUTPUTS
     PSCustomObject
             
 .INPUTS
     Array
             
 .Link
     N/A
         
 .Notes
     NAME:      Get-LocalGroupMembership
     AUTHOR:    Francois-Xavier Cat
     WEBSITE:   www.LazyWinAdmin.com
 #>
 
 	
 	[Cmdletbinding()]
 
 	PARAM (
         [alias('DnsHostName','__SERVER','Computer','IPAddress')]
 		[Parameter(ValueFromPipelineByPropertyName=$true,ValueFromPipeline=$true)]
 		[string[]]$ComputerName = $env:COMPUTERNAME,
 		
 		[string]$GroupName = "Administrators"
 
 		)
     BEGIN{
 
     PROCESS{
         foreach ($Computer in $ComputerName){
             TRY{
                 $Everything_is_OK = $true
 
                 Write-Verbose -Message "$Computer - Testing connection..."
                 Test-Connection -ComputerName $Computer -Count 1 -ErrorAction Stop |Out-Null
 	                    
                 Write-Verbose -Message "$Computer - Querying..."
 	            $Group = [ADSI]"WinNT://$Computer/$GroupName,group"
 	            $Members = @($group.psbase.Invoke("Members"))
             CATCH{
                 $Everything_is_OK = $false
                 Write-Warning -Message "Something went wrong on $Computer"
                 Write-Verbose -Message "Error on $Computer"
         
             IF($Everything_is_OK){
                 Write-Verbose -Message "$Computer - Formatting Data"
 	            $members | ForEach-Object {
 		            $name = $_.GetType().InvokeMember("Name", 'GetProperty', $null, $_, $null)
 		            $class = $_.GetType().InvokeMember("Class", 'GetProperty', $null, $_, $null)
 		            $path = $_.GetType().InvokeMember("ADsPath", 'GetProperty', $null, $_, $null)
 		
 		            if ($path -like "*/$Computer/*"){
 			            $Type = "Local"
 			            }
 		            else {$Type = "Domain"
 		            }
 
 		            $Details = "" | Select-Object ComputerName,Account,Class,Group,Path,Type
 		            $Details.ComputerName = $Computer
 		            $Details.Account = $name
 		            $Details.Class = $class
                     $Details.Group = $GroupName
 		            $details.Path = $path
 		            $details.Type = $type
 		
                     $Details
 	            }
 
`

