---
Author: cody bunch
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3168
Published Date: 2012-01-16t23
Archived Date: 2012-01-21t08
---

# the powershell talk xen2 - 

## Description

the powershell talk – demo 2, vm easy bake, xenserver

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 Get-Credential | connect-Xenserver -Url http://XenServer_URL/sdk
 
 Create-XenServer:VM -NameLabel "Dave" -VCPUsAtStartup 1 -MemoryDynamicMax 536870912 -MemoryStaticMax 536870912 -MemoryDynamicMin 536870912 -MemoryStaticMin 536870912 -MemoryTarget 536870912
 
 Get-XenServer:VM -name "Dave" | fl * | more
 
 Get-XenServer:vm -name "Dave" | Set-XenServer:VM.MemoryDynamicMax -DynamicMax 268435456
 Get-XenServer:vm -name "Dave" | Set-XenServer:VM.MemoryDynamicMin -DynamicMin 268435456
 Get-XenServer:vm -name "Dave" | Set-XenServer:VM.MemoryStaticmin -StaticMin 268435456
 Get-XenServer:vm -name "Dave" | Set-XenServer:VM.MemoryStaticMax -Value 268435456
 
 Get-XenServer:vm -name "Dave" | Destroy-XenServer:VM
`

