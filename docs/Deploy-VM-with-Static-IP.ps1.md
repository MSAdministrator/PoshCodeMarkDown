---
Author: nedko nedev
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 2453
Published Date: 
Archived Date: 2011-01-26t17
---

# deploy vm with static ip - 

## Description

deploying a vm with static ip in 3 steps

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $custSpec = New-OSCustomizationSpec -Type NonPersistent -OSType Windows `
     -OrgName �My Organization� -FullName �MyVM� -Domain �MyDomain� `
     �DomainUsername �user� �DomainPassword �password�
 
 $custSpec | Get-OSCustomizationNicMapping | Set-OSCustomizationNicMapping `
     -IpMode UseStaticIP -IpAddress 192.168.121.228 -SubnetMask 255.255.248.0 `
     -Dns 192.168.108.1 -DefaultGateway 192.168.108.1
 
 New-VM -Name �MyNewVM� -Template $template -VMHost $vmHost `
     -OSCustomizationSpec $custSpec
`

