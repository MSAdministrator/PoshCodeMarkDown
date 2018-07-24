---
Author: steven murawski http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 684
Published Date: 
Archived Date: 2009-01-05t17
---

# show-adgroupmembership - 

## Description

this script uses the quest ad cmdlets to retrieve ad groups from an ldap search root and maps their membership (shows nested groups using doug finke’s show-netmap scripts that leverage the microsoft research netmap project.  improvements or suggestions welcomed!

## Comments



## Usage



## TODO



## script

`new-sourcetarget`

## Code

`#
 #
 
 
 param([string]$SearchRoot= 'yourdomain.local/usersOU')
 
 Function New-SourceTarget ($s,$t) {
 	New-Object PSObject |
 		Add-Member -pass noteproperty source $s |
 		Add-Member -pass noteproperty target $t
 }
 
 $groups = Get-QADGroup -GroupType Security -SearchRoot $SearchRoot
 
 [string[]]$GroupNames = $groups | foreach {$_.name}
 
 $sources = @()
 
 foreach ($group in $groups)
 {
 	$name = $group.name
 	foreach ($member in $group.members)
 	{
 		$SubGroupName = $member -replace '^CN=(.+?),OU=.*', '$1'
 		if ($GroupNames -contains $SubGroupName)
 		{
 			$sources += New-SourceTarget $SubGroupName $name
 		}
 	}
 	
 }
 
 . c:\scripts\powershell\Show-NetMap
 
 $sources | Show-NetMap
`

