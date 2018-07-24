---
Author: andy stumph
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1068
Published Date: 2010-04-30t08
Archived Date: 2015-03-11t07
---

# new-adsubnet - 

## Description

this script will create a new subnet in ad sites and services given the subnet, site name and location.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 param ($Subnet, $SiteName, $Location, [switch]$Help)
 
 function Help
 {
 ""
 Write-Host "Usage: .\New-ADSubnet.ps1 -Help" -foregroundcolor Yellow
 Write-Host "Usage: .\New-ADSubnet.ps1 <Subnet> <SiteName> <Location>" -foregroundcolor Yellow
 Write-Host "Ex: .\New-ADSubnet.ps1 10.150.0.0/16 Default-First-Site-Name USA/KY/Louisville" -foregroundcolor Yellow
 ""
 Break
 }
 
 if ($Help) {Help}
 if ($Subnet -eq $Null) {Write-Host "Please provide a Subnet!" -fore Red; Help}
 if ($Location -eq $Null) {Write-Host "Please provide a Location!" -fore Red; Help}
 if ($SiteName -eq $Null) {Write-Host "Please provide a Site Name!" -fore Red; Help}
 
 if ($SiteName -like "CN=*")
 {
 	$SiteNameRDN = $SiteName
 }
 else
 {
 	$SiteNameRDN = "CN=$($SiteName)"
 }
 
 $Description = $Subnet
 
 $RootDSE = [ADSI]"LDAP://RootDSE"
 $ConfigurationNC = $RootDSE.configurationNamingContext
 
 $SubnetRDN = "CN=$($Subnet)"
 $Description = $Subnet
 
 $SiteDN = "$($SiteNameRDN),CN=Sites,$($ConfigurationNC)"
 $SubnetsContainer = [ADSI]"LDAP://CN=Subnets,CN=Sites,$($ConfigurationNC)"
 
 $NewSubnet = $SubnetsContainer.Create("subnet",$SubnetRDN)
 
 $NewSubnet.Put("siteObject", $SiteDN)
 $NewSubnet.Put("description", $Description)
 $NewSubnet.Put("location", $Location)
 
 trap {Continue}
 $NewSubnet.SetInfo()
 
 if (!$?)
 {
 ""
 Write-Host "An Error has Occured! Please Validate Your Input." -foregroundcolor Red
 Write-Host "Error Message:" -foregroundcolor Red
 $Error[0].Exception
 ""
 }
 else
 {
 ""
 Write-Host "Subnet Created Successfully." -foregroundcolor Green
 ""
 }
`

