---
Author: jan egil ring
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1589
Published Date: 
Archived Date: 2010-01-25t01
---

# gppreferencesprinters - gppreferencesprinters.psm1

## Description

contains a function for adding a shared printer connection, and a function for retrieving shared printer connections from a specified group policy object.

## Comments



## Usage



## TODO



## script

`add-gppreferencesprinter`

## Code

`#
 #
 ###########################################################################"
 #
 #
 #
 #
 #
 #
 ###########################################################################"
 
 Import-Module SDM-GroupPolicy
 
 function Add-GPPreferencesPrinter {
 ################################################################
 #.Synopsis
 #.Parameter GPOName
 ################################################################
 param( [Parameter(Mandatory=$true)][string]$GPOName,[Parameter(Mandatory=$true)][string]$Printername,[Parameter(Mandatory=$true)][string]$Sharepath,[Parameter(Mandatory=$true)][string]$Action,[Parameter(Mandatory=$false)][string]$Default,[Parameter(Mandatory=$false)][string]$DefaulIfNoLocalPrinters)
 
 $domain = [System.DirectoryServices.ActiveDirectory.Domain]::getcurrentdomain() 
 $domainname = $domain.name
 $gpo = Get-SDMgpobject -gpoName "gpo://$domainname/$GPOName" -openByName
 $container = $gpo.GetObject("User Configuration/Preferences/Control Panel Settings/Printers/Shared printer")
 $add = $container.Settings.AddNew("$Printername")
 $add.put("Action",[GPOSDK.EAction]$Action)
 $add.put("Share path",$Sharepath)
 $add.put("Default",$Default)
 $add.put("Default if no local printers",$DefaulIfNoLocalPrinters)
 $add.Save()
 }
 
 function Get-GPPreferencesPrinter {
 ################################################################
 #.Synopsis
 #.Parameter GPOName
 ################################################################
 param(
 [Parameter(Mandatory=$true)]
 [string]$GPOName
 )
 
 $domain = [System.DirectoryServices.ActiveDirectory.Domain]::getcurrentdomain() 
 $domainname = $domain.name
 $gpo = Get-SDMgpobject -gpoName "gpo://$domainname/$GPOName" -openByName
 $container = $gpo.GetObject("User Configuration/Preferences/Control Panel Settings/Printers/Shared printer")
 $setting = $container.Settings
 $printers =  @()
 $count = $setting.Count
 $counter = $count; while($counter -ge 1) {$counter = $counter - 1 
 $printer = $Setting.Item($counter)
 foreach ($i in $printer) {
 $printerinfo = @{}
 $printerinfo.Name = $i.Name
 $printerinfo.Sharepath = ($setting.Item($counter)).Get("Share path")
 $printerinfo.Action = ($setting.Item($counter)).Get("Action")
 $printerinfo.SetDefault = ($setting.Item($counter)).Get("Default")
 $printerinfo.SetDefaulIfNoLocalPrinters = ($setting.Item($counter)).Get("Default if no local printers")
 $printers += $printerinfo
 }
 }
 $printers | Select-Object @{Name="Name"; Expression = {$_.name}},@{Name="Sharepath"; Expression = {$_.sharepath}},@{Name="Action"; Expression = {$_.Action}},@{Name="SetDefault"; Expression = {$_.SetDefault}},@{Name="SetDefaulIfNoLocalPrinters"; Expression = {$_.SetDefaulIfNoLocalPrinters}} | ft Name,Sharepath,Action,SetDefault,SetDefaulIfNoLocalPrinters
 }
`

