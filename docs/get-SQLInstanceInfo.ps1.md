---
Author: rich vantrease
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3978
Published Date: 2013-02-21t01
Archived Date: 2016-03-06t01
---

# get-sqlinstanceinfo - 

## Description

r.vantrease – function to return the following; displayname, name, clustered, instanceid, fileversion, version, virtualname(if clustered), instance, port, servicestate, serviceaccount, edition, auditlevel, loginmode, physicalmemory, processors, product, productlevel, majorversion, minorversion, build, release, namedpipesenabled)

## Comments



## Usage



## TODO



## function

`get-sqlinstanceinfo2`

## Code

`#
 #
 function get-SQLInstanceInfo2
 {
 	param
 	(
 		[Parameter(Mandatory=$True)][string]$ComputerName
 	)
 	$InstanceInfos = @()
 
     $Instances = (new-object 'microsoft.sqlserver.management.smo.Wmi.ManagedComputer' "$ComputerName").Services | where-object{$_.type -eq 'SqlServer'}
 
     foreach($Instance in $Instances )
     {
         
         [psobject]$InstanceInfo = "" | Select-Object DisplayName, Name, Clustered, `
                                                         InstanceID, FileVersion, Version, `
                                                         VirtualName, Instance, Port, ServiceState, `
                                                         ServiceAccount, Edition, AuditLevel, LoginMode, `
                                                         PhysicalMemory, Processors, Product, ProductLevel, `
                                                         MajorVersion, MinorVersion, Build, Release,NamedPipesEnabled
                                                     
         write-verbose "Processing SQL Instance: $($Instance.Name)"
         
         $InstanceInfo.DisplayName    = $Instance.DisplayName
         $InstanceInfo.Name           = $Instance.Name
         $InstanceInfo.Clustered      = $Instance.AdvancedProperties['CLUSTERED'].Value
         $InstanceInfo.InstanceID     = $Instance.AdvancedProperties['INSTANCEID'].Value
         $InstanceInfo.FileVersion    = $Instance.AdvancedProperties['FILEVERSION'].Value
         $InstanceInfo.Version        = $Instance.AdvancedProperties['VERSION'].Value
         if($Instance.AdvancedProperties['VSNAME'].Value -eq ''){ $InstanceInfo.VirtualName = $(hostname) }
         else { $InstanceInfo.VirtualName    = $Instance.AdvancedProperties['VSNAME'].Value }
         if($Instance.Name.Split('$')[1] -eq $Null){ [string]$InstanceName = 'MSSQLSERVER' }
         else { [string]$InstanceName = $Instance.Name.Split('$')[1] }
         $InstanceInfo.Instance       = $InstanceName
         $InstanceInfo.Port           = $Instance.Parent.ServerInstances[$InstanceName].ServerProtocols['Tcp'].IPAddresses['IPALL'].IPAddressProperties['TcpPort'].Value
         $InstanceInfo.NamedPipesEnabled = $Instance.Parent.ServerInstances[$InstanceName].ServerProtocols['Np'].IsEnabled
         $InstanceInfo.ServiceAccount = $Instance.ServiceAccount
         $InstanceInfo.ServiceState   = $Instance.ServiceState
         
         if($InstanceInfo.Clustered){ $SQLInstanceName = "$($InstanceInfo.VirtualName),$($InstanceInfo.Port)" }
         else { $SQLInstanceName = "$ComputerName,$($InstanceInfo.Port)" }
         
         write-verbose "SQL InstanceName: $SQLInstanceName"
         
         [System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.Smo") |out-null
 
         $SQL = new-object "Microsoft.SQLServer.Management.SMO.Server" $SQLInstanceName
 
         $InstanceInfo.Edition        = $SQL.Edition
         $InstanceInfo.AuditLevel     = $SQL.AuditLevel
         $InstanceInfo.LoginMode      = $SQL.LoginMode
         $InstanceInfo.PhysicalMemory = $SQL.PhysicalMemory
         $InstanceInfo.Processors     = $SQL.Processors
         $InstanceInfo.Product        = $SQL.Product
         $InstanceInfo.ProductLevel   = $SQL.ProductLevel
         $InstanceInfo.MajorVersion   = $SQL.Version.Major
         $InstanceInfo.MinorVersion   = $SQL.Version.Minor
         $InstanceInfo.Build          = $SQL.Version.Build
         
         if($SQL.Version.Major -eq 10)
         {
             switch($SQL.Version.Minor)
             {
                 0
                 {
                     $InstanceInfo.Release = 'Gold'
                 }
                 50
                 {
                     $InstanceInfo.Release = 'R2'
                 }
                 else
                 {
                     throw "Could not convert minor version into release.  Version number $($SQL.Versioin.Minor)"
                 }
             }
         } else { $InstanceInfo.Release = 'Gold' }
          
         $InstanceInfos += $InstanceInfo
         
     }
 
     write-verbose "Showing results"
     $InstanceInfos
 }
`

