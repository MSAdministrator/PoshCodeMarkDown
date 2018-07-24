---
Author: cody bunch
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1089
Published Date: 2009-05-10t14
Archived Date: 2014-10-23t14
---

# powershell talk xen1 - 

## Description

the powershell talk, demo 1 – xenserver

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 Get-Credential | connect-Xenserver -Url http://XenServer_URL/sdk
 
 Create-XenServer:Network -NameLabel "Test Network"
 
 Get-XenServer:Network -NameFilter "Test Network" | Set-XenServer:Network.NameDescription "This is the test network for the XenServer Demo"
 
 Get-XenServer:Network -NameFilter "Test Network" | Destroy-XenServer:Network
`

