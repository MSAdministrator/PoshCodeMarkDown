---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 929
Published Date: 
Archived Date: 2009-03-15t01
---

# ontapi module - 

## Description

the beginnings of a module to manage a netapp san

## Comments



## Usage



## TODO



## script

`new-ntapserver`

## Code

`#
 #
 
 
 $OnTapDll = (resolve-path $PSScriptRoot\ManageOntap.dll).Path
 [System.Reflection.Assembly]::LoadFile($onTapDll)
 
 
 function New-NTAPServer {
 param
 (
 [Parameter( Mandatory=$true,
             Position=0,
             ValueFromPipeline = $true)]
 [string]
 $filer,
 
 [Parameter( Mandatory=$true,
             Position=1)]
 [System.Management.Automation.PSCredential]
 $credential
 
 )
 
 $username = $credential.GetNetworkCredential().UserName
 $password = $credential.GetNetworkCredential().Password
 $domain   = $credential.GetNetworkCredential().Domain
 if ($domain) {$username = "$domain\$username"}
 
 $NtapServer = New-Object NetApp.Manage.NaServer($filer,1,0)                                                                                      
 $NtapServer.SetAdminUser($username,$password)
 
 $NaElement = New-Object Netapp.Manage.NaElement("system-get-info") 
 [xml]$sysinfo = $NtapServer.InvokeElem($NaElement)       
 $sysinfo.results."system-info" | out-host
 return $NtapServer
 }
 
 
 Function Get-NTAPLun {
 param
 (
 [Parameter( Mandatory = $true,
             Position  = 0)]
 [NetApp.Manage.NaServer]
 $server,
 
 [Parameter( Mandatory=$false,
             Position=1,
             ValueFromPipeline = $true,
             ValueFromPipelineByPropertyName = $true)]
 [Alias("Path")]
 [String]
 $name = "*"
 )
 
 $NaElement = New-Object Netapp.Manage.NaElement("lun-list-info")
 [xml]$result = $server.InvokeElem($NaElement)
 return $result.results.luns."lun-info" | where {$_.path -like $name}
 }
 
 
 Function Set-NTAPLun {
 param
 (
 [Parameter( Mandatory = $true,
             Position  = 0)]
 [NetApp.Manage.NaServer]
 $server,
 
 [Parameter( Mandatory=$false,
             Position=1,
             ValueFromPipeline = $true,
             ValueFromPipelineByPropertyName = $true)]
 [Alias("Path")]
 [String]
 $name ,
 
 [Parameter( Mandatory=$false,
             Position=2,
             ValueFromPipeline = $false)]
 [int64]
 $size,
 
 [Parameter( Mandatory=$false,
             Position=3)]
 [Alias("description")]
 [string]
 $comment
 
 )
 
 if($comment) {
 $NaElement = New-Object Netapp.Manage.NaElement("lun-set-comment")
 $NaElement.AddNewChild('path',$name)
 $NaElement.AddNewChild('comment',$comment)
 [xml]$result = $server.InvokeElem($NaElement)
 $result.results.status
 }
 
 
 if ($size) {
 $NaElement = New-Object Netapp.Manage.NaElement("lun-resize")
 $NaElement.AddNewChild('path',$name)
 $NaElement.AddNewChild('size',$size)
 [xml]$result = $server.InvokeElem($NaElement)
 $result.results.status
 
 }
 
 
 }
 
 
 Function New-NTAPLun {
 param
 (
 [Parameter( Mandatory = $true,
             Position  = 0)]
 [NetApp.Manage.NaServer]
 $server,
 
 [Parameter( Mandatory=$false,
             Position=1,
             ValueFromPipeline = $true,
             ValueFromPipelineByPropertyName = $true)]
 [Alias("Path")]
 [String]
 $name ,
 
 [Parameter( Mandatory=$false,
             Position=2,
             ValueFromPipeline = $false)]
 [int64]
 $size,
 
 [Parameter( Mandatory=$false,
             Position=3,
             ValueFromPipeline = $false)]
 [Alias("description")]
 [string]
 $comment
 )
 
 $NaElement = New-Object Netapp.Manage.NaElement("lun-create-by-size")
 $NaElement.AddNewChild('path',$name)
 $NaElement.AddNewChild('size',$size)
 [xml]$result = $Server.InvokeElem($NaElement)
 $result.results.status
 
 $NaElement = New-Object Netapp.Manage.NaElement("lun-set-comment")
 $NaElement.AddNewChild('path',$name)
 $NaElement.AddNewChild('comment',$size)
 [xml]$result = $Server.InvokeElem($NaElement)
 $result.results.status
 
 
 
 }
 
 Function Get-NTAPInitiator {
 param
 (
 [Parameter( Mandatory = $true,
             Position  = 0)]
 [NetApp.Manage.NaServer]
 $server,
 
 [Parameter( Mandatory=$false,
             Position=1,
             ValueFromPipeline = $true,
             ValueFromPipelineByPropertyName = $true)]
 [Alias("Initiator")]
 [String]
 $name = "*"
 )
 
 $NaElement = New-Object Netapp.Manage.NaElement("iscsi-initiator-list-info")
 [xml]$result = $Server.InvokeElem($NaElement)
 return $result.results."iscsi-initiator-list-entries"."iscsi-initiator-list-entry-info" |
 where {$_."initiator-nodename" -like $name}
 
 }
 
 Function Get-NTAPVolume {
 param
 (
 [Parameter( Mandatory = $true,
             Position  = 0)]
 [NetApp.Manage.NaServer]
 $server,
 
 [Parameter( Mandatory=$false,
             Position=1,
             ValueFromPipeline = $true,
             ValueFromPipelineByPropertyName = $true)]
 [Alias("Path")]
 [String]
 $name = "*"
 )
 
 $NaElement = New-Object Netapp.Manage.NaElement("volume-list-info")
 [xml]$r = $server.InvokeElem($NaElement)
 return $r.results.volumes."volume-info" |
 where {$_.name -like $name}
 
 }
 
 Function Set-NTAPVolume {
 param
 (
 
 [Parameter( Mandatory=$true,
             Position=0)]
 [NetApp.Manage.NaServer]
 $server,
 
 [Parameter( Mandatory=$true,
             Position=1,
             ValueFromPipeline = $true,
             ValueFromPipelineByPropertyName = $true)]
 [String]
 $name,
 
 [Parameter( Mandatory=$false,
             Position=2,
             ValueFromPipelineByPropertyName = $true)]
 [int64]
 $size,
 
 [Parameter( Mandatory=$false,
             Position=3)]
 [ValidateSet("none", "file", "volume")]
 [String]
 $spaceReserveType = "none",
 
 [Parameter( Mandatory=$true,
             Position=4,
             ValueFromPipeline = $true,
             ValueFromPipelineByPropertyName = $true)]
 [String]
 $newName
 )
 
 if ($newname) {
 $NaElement = New-Object Netapp.Manage.NaElement("volume-rename")
 $NaElement.AddNewChild('volume',$name)
 $NaElement.AddNewChild('new-volume-name',$newname)
 [xml]$result = $Server.InvokeElem($NaElement)
 $result.results.status
 }
 
 if ($size) {
 $NaElement = New-Object Netapp.Manage.NaElement("volume-size")
 $NaElement.AddNewChild('volume',$name)
 $NaElement.AddNewChild('new-size',$size)
 [xml]$result = $Server.InvokeElem($NaElement)
 $result.results.status
 }
 
 if ($spaceReserveType) {
 $NaElement = New-Object Netapp.Manage.NaElement("volume-set-option")
 $NaElement.AddNewChild('option-name','guarantee')
 $NaElement.AddNewChild('option-value',$spaceReserveType)
 [xml]$result = $Server.InvokeElem($NaElement)
 $result.results.status
 }
 
 }
 
 Function New-NTAPVolume {
 
 param
 (
 
 [Parameter( Mandatory = $true,
             Position  = 0)]
 [NetApp.Manage.NaServer]
 $server,
 
 [Parameter( Mandatory=$false,
             Position=1,
             ValueFromPipeline = $true,
             ValueFromPipelineByPropertyName = $true)]
 [String]
 $name = "*",
 
 [Parameter( Mandatory=$false,
             Position=2,
             ValueFromPipelineByPropertyName = $true)]
 [int64]
 $size,
 
 [Parameter( Mandatory=$false,
             Position=3,
             ValueFromPipeline = $true)]
 [Alias("Path")]
 [String]
 $containingAggregate,
 
 [Parameter( Mandatory=$false,
             Position=4)]
 [ValidateSet("none", "file", "volume")]
 [String]
 $spaceReserveType = "none"
 
 )
 
 $NaElement = New-Object Netapp.Manage.NaElement("volume-create")
 $NaElement.AddNewChild('volume',$name)
 $NaElement.AddNewChild('size',$size)
 $NaElement.AddNewChild('containing-aggr-name',$containingaggregate)
 $NaElement.AddNewChild('space-reserve',$spaceReserveType)
 [xml]$result = $Server.InvokeElem($NaElement)
 $result.results.status
 
 }
 
 Function Get-NTAPAggregate {
 param
 (
 [Parameter( Mandatory = $true,
             Position  = 0)]
 [NetApp.Manage.NaServer]
 $server,
 
 [Parameter( Mandatory=$false,
             Position=1,
             ValueFromPipeline = $true,
             ValueFromPipelineByPropertyName = $true)]
 [Alias("Path")]
 [String]
 $name = "*"
 )
 
 $NaElement = New-Object Netapp.Manage.NaElement("aggr-list-info")
 [xml]$r = $server.InvokeElem($NaElement)
 return $r.results.aggregates."aggr-info" |
 where {$_.name -like $name}
 }
`

