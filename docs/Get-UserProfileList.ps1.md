---
Author: chris rakowitz
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5954
Published Date: 2015-07-30t17
Archived Date: 2015-08-02t00
---

# get-userprofilelist - 

## Description

used to get a list of user profiles from a target pc.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 #/    \/    \/    \/    \/    \/    \/    \/    \/    \/    \/    \/    \
 #|----
 #|----@Author:  Chris Rakowitz
 #|----@Purpose: Returns a list of User Profiles from a Target Machine.
 #|----				Pressing Enter without entering a PC Name or IP will
 #|----				generate the information for the local machine.
 #|----            Usage:  .\Single-GetUserList.ps1 <PCNAME or IP>
 #|----@Date: April 15, 2015
 #|----@Version: 1
 #|----
 #\    /\    /\    /\    /\    /\    /\    /\    /\    /\    /\    /\    /
 
 [cmdletbinding()]
 param
 (
 	[parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
 	$ComputerName = $env:computername
 )        
 
 $Profiles = Get-WmiObject -Class Win32_UserProfile -Computer $ComputerName -ea 0
 foreach ($Profile in $Profiles)
 {
 	if($Profile.localpath -like 'C:\Windows\*') 
 	{
 		continue
 	}
 	else
 	{
 		$objSID = New-Object System.Security.Principal.SecurityIdentifier($Profile.sid)
 		$objuser = $objSID.Translate([System.Security.Principal.NTAccount])
 
 		$OutputObj = New-Object -TypeName PSobject
 		$OutputObj | Add-Member -MemberType NoteProperty -Name ComputerName -Value $Computer.toUpper()
 		$OutputObj | Add-Member -MemberType NoteProperty -Name ProfileName -Value $objuser.value
 		$OutputObj | Add-Member -MemberType NoteProperty -Name ProfilePath -Value $Profile.localpath
 		
 		$OutputObj
 	}
 }
`

