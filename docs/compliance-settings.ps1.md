---
Author: hal rottenberg
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5245
Published Date: 2014-06-18t12
Archived Date: 2014-06-25t07
---

# compliance settings - 

## Description

find matching members in a local group

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 $ChildGroups = "Domain Admins", "Group Two"
 $LocalGroup = "Administrators"
 
 $MemberNames = @()
 foreach ( $Server in $Servers ) {
 	$Group= [ADSI]"WinNT://$Server/$LocalGroup,group"
 	$Members = @($Group.psbase.Invoke("Members"))
 	$Members | ForEach-Object {
 		$MemberNames += $_.GetType().InvokeMember("Name", 'GetProperty', $null, $_, $null)
 	} 
 	$ChildGroups | ForEach-Object {
 		$output = "" | Select-Object Server, Group, InLocalAdmin
 		$output.Server = $Server
 		$output.Group = $_
 		$output.InLocalAdmin = $MemberNames -contains $_
 		Write-Output $output
 	}
 }
`

