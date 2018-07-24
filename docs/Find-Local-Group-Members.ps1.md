---
Author: hal rottenberg
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 905
Published Date: 
Archived Date: 2009-06-07t10
---

# find local group members - 

## Description

amended line $childgroups = “domain admins”, “group two” to clear bug from my previous post

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 $ChildGroups = "Domain Admins", "group two"
 $LocalGroup = "Administrators"
 $error.clear()
 $MemberNames = @()
  $Servers = Get-Content serverlist.txt
 foreach ( $Server in $Servers ) {
 	$MemberNames = @()
 	$Group= [ADSI]"WinNT://$Server/$LocalGroup,group"
 	$Members = @($Group.psbase.Invoke("Members"))
 	$Members | ForEach-Object {
 		$MemberNames += $_.GetType().InvokeMember("Name", 'GetProperty', $null, $_, $null)
 	
 	} 
          $errCount = $error.count
 	$ChildGroups | ForEach-Object {
 		$output = "" | Select-Object Server, Group, InLocalAdmin
 		$output.Server = $Server
 		$output.Group = $_
 		If ($errCount -gt 0){$output.InLocalAdmin = "Connection Error!"}ELSE{$output.InLocalAdmin = $MemberNames -contains $_}
 		Write-Output $output
 	}
 $error.clear()
 }
`

