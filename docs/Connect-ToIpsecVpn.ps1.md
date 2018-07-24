---
Author: adam bertram
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4721
Published Date: 2013-12-19t14
Archived Date: 2013-12-22t06
---

# connect-toipsecvpn - 

## Description

this script uses the cisco ipsec vpn client to connect to a vpn gateway and immediately rdp to a device.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Read-Host -AsSecureString | ConvertFrom-SecureString | Out-File encrypted_password.txt
 $vpn_profile = 'Profile name'
 
 $username = 'username'
 $enc_password = (gc .\encrypted_password.txt | ConvertTo-SecureString)
 $credentials = new-object -typename System.Management.Automation.PSCredential -argumentlist $username,$enc_password
 $password = $credentials.GetNetworkCredential().Password
 
 Set-Location 'c:\Program Files (x86)\Cisco Systems\VPN Client'
 .\vpnclient.exe connect $vpn_profile user $username pwd $password
 
 #.\vpnclient.exe disconnect
 
 mstsc /v:HOSTNAME /multimon
`

