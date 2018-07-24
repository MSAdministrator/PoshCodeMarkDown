---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 597
Published Date: 
Archived Date: 2008-09-26t19
---

# sms.psm1 - 

## Description

module with functions to make it easy to query the sms provider directly through wmi

## Comments



## Usage



## TODO



## function

`get-smswmi`

## Code

`#
 #
 #
 
 
 
 [string]$default_wmi_provider_server = 'servername'
 [string]$default_site = 'S00'
 
 
 #
 
 function Get-SmsWmi {
   param([string]$class = $(Throw @"
 `t
 ERROR: You must enter a class name or nickname.
 `t
 Valid nicknames are:
 `t
   AddRemovePrograms
   AdStatus
   Advertisement
   Collection
   ComputerSystem
   DistributionPoint
   LogicalDisk
   MembershipRule
   NetworkAdapter
   NetworkAdapterConfiguration
   OperatingSystem
   Package
   PackageStatus
   Program
   Query
   Server
   Service
   Site
   StatusMessage
   System
   WorkstationStatus
   User
 `t
 Note: You only need to type as many characters as necessary to be unambiguous.
 `t
 "@),
         [string]$filter = $null,
         [string]$computer_name = $default_wmi_provider_server,
         [string]$site = $default_site)
 
   $classes = @{'collection' = 'SMS_Collection';
                'package' = 'SMS_Package';
                'program' = 'SMS_Program';
                'system' = 'SMS_R_System';
                'server' = 'SMS_SystemResourceList';
                'advertisement' = 'SMS_Advertisement';
                'query' = 'SMS_Query';
                'membershiprule' = 'SMS_CollectionMembershipRule';
                'statusmessage' = 'SMS_StatusMessage';
                'site' = 'SMS_Site';
                'user' = 'SMS_R_User';
                'pkgstatus' = 'SMS_PackageStatus';
                'addremoveprograms' = 'SMS_G_System_ADD_REMOVE_PROGRAMS';
                'computersystem' = 'SMS_G_System_COMPUTER_SYSTEM';
                'operatingsystem' = 'SMS_G_System_OPERATING_SYSTEM';
                'service' = 'SMS_G_System_SERVICE';
                'workstationstatus' = 'SMS_G_System_WORKSTATION_STATUS';
                'networkadapter' = 'SMS_G_System_NETWORK_ADAPTER';
                'networkadapterconfiguration' = ('SMS_G_System_NETWORK_' +
                                                 'ADAPTER_CONFIGURATION');
                'logicaldisk' = 'SMS_G_System_LOGICAL_DISK';
                'distributionpoint' = 'SMS_DistributionPoint';
                'adstatus' = 'SMS_ClientAdvertisementStatus'}
 
   $matches = @();
 
   foreach ($class_name in @($classes.Keys | sort)) {
     if ($class_name.StartsWith($class.ToLower())) {
       $matches += $classes.($class_name)
     }
   }
 
   if ($matches.Count -gt 1) {
     Write-Error @"
 `t
 Warning: Class provided matches more than one nickname.
 `t
 Type 'Get-SMSWmi' with no parameters to see a list of nicknames.
 `t
 "@
     $class = $matches[0]
   } elseif ($matches.Count -eq 1) {
     $class = $matches[0]
   }
 
   $query = "Select * From $class"
 
   if ($filter) {
     $query += " Where $filter"
   }
 
   $namespace = 'root\sms\site_' + $site
   gwmi -ComputerName $computer_name -Namespace $namespace -Query $query
 }
 
 
 #
 #
 
 function Find-SmsID {
   param([switch]$advertisement_id,
         [switch]$collection_id,
         [switch]$resource_id,
         [switch]$package_id,
         [string]$id)
   $Class = ''
   $Type = ''
 
     if ($advertisement_id) {
       $type = 'AdvertisementID'
       $class = 'Advertisement'
     } elseif ($collection_id) {
       $type = 'CollectionID'
       $class = 'Collection'
     } elseif ($package_id) {
       $type = 'PackageID'
       $class = 'Package'
     } elseif ($resource_id) {
       $type = 'ResourceID'
       $class = 'System'
     } else {
       Throw @"
 `t
 You must specify an ID type.  Valid switches are:
 `t
 `t-advertisement_id
 `t-collection_id
 `t-package_id
 `t-resource_id
 `t
 USAGE: Find-SmsID <Type> <ID>
 `t
 "@
     }
 
   if ($resource_id) {
     trap [System.Exception] {
       Write-Output "`nERROR: Invalid Input for ResourceID!`n"
       break
     }
 	
     $type = 'ResourceID'
     $class = 'System'
   } else{
     if ($id -notmatch '^[a-zA-Z0-9]{8}$') {
       Throw "`n`t`nERROR: Invalid ID format.`n`t`n"
     }
   }
 
   Get-SmsWmi $class "$type = `"$ID`""
 }
`

