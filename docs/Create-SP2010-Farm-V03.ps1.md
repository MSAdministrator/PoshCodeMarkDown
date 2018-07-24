---
Author: jos verlinde
Publisher: 
Copyright: 
Email: 
Version: 0.3
Encoding: utf-8
License: cc0
PoshCode ID: 4572
Published Date: 2013-10-31t10
Archived Date: 2013-11-04t00
---

# create sp2010 farm v03 - 

## Description

create a sharepoint 2010 farm

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Param (	[String] $Farm		= "SP2010",
 	[String] $SQLServer 	= $env:COMPUTERNAME,
 	[String] $Passphrase	= "pass@word1",
 	[int]	 $CAPort	    = 26101	,
     [switch] $Force         = $false )
     
 
 
 New-ItemProperty HKLM:\System\CurrentControlSet\Control\Lsa -Name "DisableLoopbackCheck"  -value "1" -PropertyType dword
 
 
 
 
 
 $SecPhrase=ConvertTo-SecureString  $Passphrase �AsPlaintext �Force
 $Passphrase = $null
 
 $cred_farm = $host.ui.PromptForCredential("FARM Setup", "SP Farm Account (SP_farm)", "contoso\sp_farm", "NetBiosUserName" )
 
 
 
 
 
 New-SPConfigurationDatabase �DatabaseName �$FARM-Config� �DatabaseServer $SQLServer �AdministrationContentDatabaseName �$FARM-Admin-Content� �Passphrase $SecPhrase �FarmCredentials $Cred_Farm
 
 New-SPCentralAdministration -Port $CAPort -WindowsAuthProvider "NTLM"
 
 Install-SPApplicationContent 
 
 
 Initialize-SPResourceSecurity
 
 
 If ( $Force ) {
     $Features = Install-SPFeature �AllExistingFeatures -force
 } else {
     $Features = Install-SPFeature �AllExistingFeatures 
 }    
 $Features 
 
 
 
 Start-Process "http://$($env:COMPUTERNAME):$CAPort"
 
 Start-Process "http://$($env:COMPUTERNAME):$CAPort/_admin/adminconfigintro.aspx?scenarioid=adminconfig&welcomestringid=farmconfigurationwizard_welcome"
 
 
 ##@@ Todo - Run Farm Wizard or better yet create required service applications (minimal - normal - all template)
`

