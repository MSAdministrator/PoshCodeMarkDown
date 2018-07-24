---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 3.5
Encoding: ascii
License: cc0
PoshCode ID: 921
Published Date: 
Archived Date: 2009-03-11t04
---

# ontapsdk_discoverability - 

## Description

a couple functions to aid in the discoverability of the ontap sdk. functions requires the netapp ontap sdk v3.5 which can be found http

## Comments



## Usage



## TODO



## function

`get-naapi`

## Code

`#
 #
 
 #########################################################################
 #
 #
 #
 Function Get-NaAPI {
     Param(
         [NetApp.Manage.NaServer]$NaServer,
         [string]$Name
     )
     Begin {
         $APIList = @()
     }
     Process {
         $NaElement = New-Object Netapp.Manage.NaElement("system-api-list")
         $apis = $NaServer.InvokeElem($naelement).GetChildByName("apis")
         Foreach ($api in $apis.GetChildren()) {
            $APIList +=  $api.getchildcontent("name")
         }
     }
     End {
         if ($Name) {
            return $APIList | Where-Object {$_ -match $name}
         } else {
             return $APIList
         }
     }
 }
 
 #########################################################################
 #
 #
 #
 #
 Function Get-NaAPIElements {
     Param(
         [NetApp.Manage.NaServer]$NaServer,
         [string[]]$API,
         [switch]$Output
     )
     Process {
         $e = New-Object NetApp.Manage.NaElement("api-list")
         Foreach ($A in $API) {
             $e.AddNewChild('api-list-info',$A)
         }
         $NaElement = New-Object NetApp.Manage.NaElement("system-api-get-elements")
         $NaElement.AddChildElement($e)
         $apilist = $NaServer.InvokeElem($naelement).getchildbyname("api-entries")
         Foreach ($A in $apiList.getchildbyname("system-api-entry-info")) {
             $Obj = "" | Select API, ElementName, ElementType, Optional, AllowNull 
             $N = $A.GetChildContent("name")
             foreach ($e in $a.getchildbyname("api-elements").getchildren()) {
                 $obj.API = $N
                 $obj.ElementName = $e.getchildcontent('name')
                 
                 switch ($e.getchildcontent('type')) {
                     "string"  {$obj.ElementType = "String"}
                     "integer" {$obj.ElementType = "int"}
                     "boolean" {$obj.ElementType = "Bool"}
                     Default   {
                         $obj.ElementType = $_
                         $objElem = $true
                         }
                 }    
                 
                 IF (!$e.getchildbyname('is-optional')) {
                    $Obj.Optional = $False
                 } Else {
                    $Obj.Optional = $true
                 }
                 IF (!$e.getchildbyname('is-output')) {
                    $objOutput = $False
                 } Else {
                    $objOutput = $true
                 }
                 IF (!$e.getchildbyname('is-optional')) {
                    $Obj.Optional = $False
                 } Else {
                    $Obj.Optional = $true
                 }
                 IF (
                     (!$e.getchildbyname('is-nonempty')) -or `
                     $e.GetChildContent('is-nonempty') -match "false"
                    ) {
                    $Obj.AllowNull = $True
                 } Else {
                    $Obj.AllowNull = $False
                 }
                 
                 Switch ($objOutput) {
                     {($Output -eq $true) -and ($_ -eq $true)}
                         {write-output $obj}
                     {($Output -eq $false) -and ($_ -eq $false)}
                         {write-output $obj}
                 }
             }
         }
     }
 }
 
 #########################################################################
 #
 #
 Function Get-NaType {
     Param(
         [NetApp.Manage.NaServer]$NaServer,
         [string]$Type
     )
     process {
         $NaElement = New-Object NetApp.Manage.NaElement("system-api-list-types")
         $results = $NaServer.InvokeElem($NaElement).GetChildByName("type-entries").getchildren()  
         If ($type) {
             $results = $results | ?{$_.GetChildContent("name") -eq $type}
         }
         $results.GetChildByName("type-elements").getchildren() | Select @{
                     n='name'
                     E={$_.GetChildContent("name")}
                 }, @{
                     n='type'
                     E={$_.GetChildContent("type")}
                 }, @{
                     n='optional'
                     E={$_.GetChildContent("is-optional")}
                 }
 
     }
     
 }
`

