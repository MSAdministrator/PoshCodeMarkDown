---
Author: leon scheltema
Publisher: 
Copyright: 
Email: 
Version: 5.1
Encoding: ascii
License: cc0
PoshCode ID: 4335
Published Date: 2015-07-26t11
Archived Date: 2015-12-13t00
---

# relocate vapp - 

## Description

script changes the storage profile on al vms in the vapp, this triggers a relocate action per vm moving it to a other datastore or other storage system.

## Comments



## Usage



## TODO



## function

`get-vcloud51`

## Code

`#
 #
 function Get-vCloud51($href)
 {
  $request = [System.Net.HttpWebRequest]::Create($href)
  $request.Accept = "application/*+xml;version=5.1"
  $request.Headers.add("x-vcloud-authorization",$global:DefaultCIServers[0].SessionId)
  $response = $request.GetResponse()
  $streamReader = new-object System.IO.StreamReader($response.getResponseStream())
  $xmldata = $streamreader.ReadToEnd()
  $streamReader.close()
  $response.close()
  return $xmldata
 }
  
  
 function Get-storageHref($orgVdc,$profileName)
 {
  $orgVdc51 = Get-vCloud51 $orgVdc.Href
  $storageProfileHref = $orgVdc51.vdc.VdcStorageProfiles.VdcStorageProfile | Where-Object{$_.name -eq "$profileName"} | foreach {$_.href}
  return $storageProfileHref
 }
  
  
 $vappName = read-host "vApp name"
 $profileName = read-host "Storage Profile"
 $orgVdcName = read-host "Org vDC Name"
  
 $orgVdc = get-orgvdc $orgVdcName
  
 
 $profileHref = "https://vCD-server/api/vdcStorageProfile/871be6dc-71f4-4745-b1f2-d6566930204c"
 
  
  
 $CIvApp = Get-CIVApp $vappName
 Foreach ($CIVM in ($CIvApp | Get-CIVM)) {
  $newSettings = $CIVM.extensiondata
  $newSettings.storageprofile.name = "$profileName"
  $newSettings.storageprofile.Href = $profileHref
  Write-Host "Changing the storage profile for $CIVM.name to $profileName"
  $newSettings.UpdateServerData()
 }
`

