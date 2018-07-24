---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3335
Published Date: 2012-04-10t09
Archived Date: 2017-04-11t01
---

# get-sqlwmi - 

## Description

the get-sqlwmi function  gets port, instance and service account wmi information for all sql instances on a computer.

## Comments



## Usage



## TODO



## script

`get-sqlwmi`

## Code

`#
 #
 try {add-type -AssemblyName "Microsoft.SqlServer.SqlWmiManagement, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91" -EA Stop}
 catch {add-type -AssemblyName "Microsoft.SqlServer.SqlWmiManagement"}
 
 #######################
 <#
 .SYNOPSIS
 Gets SQL Server WMI information.
 .DESCRIPTION
 The Get-SqlWmi function  gets port, instance and service account wmi information for all SQL instances on a computer.
 .EXAMPLE
 Get-SqlWmi "Z002"
 This command gets information for computername Z002.
 .NOTES 
 Version History 
 v1.0   - Chad Miller - Initial release 
 v1.1   - Chad Miller - Fixed instance parsing issue with pathname 
 #>
 function Get-SqlWmi
 {
     [CmdletBinding()]
     param(
     [Parameter(Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
     [ValidateNotNullorEmpty()]
     [string[]]$ComputerName
     )
 
     BEGIN {}
     PROCESS {
         foreach ($computer in $computername) {
             try {
                 $wmi = new-object "Microsoft.SqlServer.Management.Smo.Wmi.ManagedComputer" $Computer -ErrorAction 'Stop'
                 
                 $ht = @{}
                 $wmi.Services| where {$_.Type -eq 'SqlServer'} | foreach {$instance = $_.PathName -replace '[^-].+\s{1}-s',""; $ht.Add($instance,$_.ServiceAccount)}
                 
                 $wmi.ServerInstances | foreach { 
                         new-object psobject -property @{
                         ComputerName=$Computer;
                         Port=$_.ServerProtocols["Tcp"].IPAddresses["IPAll"].IPAddressProperties["TcpPort"].Value;
                         AccountName=$ht[$_.Name];
                         Instance=$_.Name }
                     }
             }
             catch {
                     try {   
                             $dmoServer = New-Object -comobject "SQLDMO.SQLServer"
                             $dmoServer.loginsecure = $true
                             $instances = $dmoServer.ListInstalledInstances($computer) | foreach {($_) }
                             $dmoServer = $null
                             $instances | where { $_ -like "$computer*"} | 
                                 foreach {
                                             $dmoServer = New-Object -comobject "SQLDMO.SQLServer"
                                             $dmoServer.loginsecure = $true
                                             $dmoServer.connect($_)
                                             new-object psobject -property @{
                                                 ComputerName=$Computer;
                                                 Port=$dmoServer.registry.tcpport;
                                                 AccountName=$dmoServer.StartupAccount;
                                                 Instance = $dmoServer.ServiceName -replace 'MSSQL\$',''
                                             }
                                             $dmoServer.close()
                                             $dmoServer = $null
                                             
                                         }
                              
                                         
                     }
                     catch {
                             new-object psobject -property @{ComputerName=$Computer;Port=$null;AccountName=$null;Instance=$null}
                     }
             }
         }
     }
     END {}
 
`

