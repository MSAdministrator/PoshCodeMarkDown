---
Author: adrianwoodrup
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3938
Published Date: 2013-02-12t13
Archived Date: 2013-02-16t03
---

# vmware ds migration - 

## Description

if you need to migrate a vm to a different datastore but do not have the vmotion license (cant migrate while the vm is powered on), this script can be scheduled to run out of hours and will move the vm’s datastore and set the disk to thin prov

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #####################################################
 #####################################################
 $Server = 'SERVERNAME'
 $DataStore = 'DATASTORE'
 $VCName = 'VCENTERSERVER'
 asnp VMware.VimAutomation.Core -ErrorAction SilentlyContinue
 Connect-VIServer $VCName -User 'USERNAME' -Password 'PASSWORD'
 $VMPower = Get-VM $Server
 If($VMPower.PowerState -eq 'PoweredOn'){
 	Get-VM $Server | Shutdown-VMGuest
 	}
 Start-Sleep -s 30
 Move-VM $Server -Datastore $DataStore -DiskStorageFormat Thin
 Disconnect-VIServer -Confirm:$False
`

