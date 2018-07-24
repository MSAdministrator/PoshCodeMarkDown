---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6014
Published Date: 2016-09-14t17
Archived Date: 2016-05-17t12
---

# cleanup ts profiles - 

## Description

this code will cleanup local profiles on a server/computer. it needs admin-rights to run.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 $BaseProfilePath="C:\Users\"
 $LocalProfileFolders=gci $BaseProfilePath
 $thishost=hostname
 $thishostADSI = [ADSI]"WinNT://$thishost,computer"  
 $LocalAccountsOnThisHost=$thishostADSI.psbase.Children | Where-Object { $_.psbase.schemaclassname -eq 'user' } | % { $_.Name }
 $ExcludedUsers= @()
 $LoggedOnUsers = @()
 $UsersToIgnore = @()
 
 $SpeedBrake=1
 
 $UserNameRegEx="\w"
 
 $ExcludedUsers="Public","ctx","svc"
 
 $LoggedOnUsers=Get-WmiObject win32_process|select name,@{n="owner";e={$_.getowner().user}} | Select-Object Owner -Unique | % { $_.owner }
 
 $UsersToIgnore+=$LoggedOnUsers
 $UsersToIgnore+=$ExcludedUsers
 $UsersToIgnore+=$LocalAccountsOnThisHost
 $UsersToIgnore=$UsersToIgnore | select -Unique
 
 foreach ($LocalProfileFolder in $LocalProfileFolders)
 {
 	sleep -m $SpeedBrake
 
 	$ThisProfileShouldBeDeleted=$True
 
 	if ($LocalProfileFolder.Name -match $UserNameRegEx)
 	{
 		foreach ($UserToIgnore in $UsersToIgnore)
 		{
 		sleep -m $SpeedBrake
 			if ($LocalProfileFolder.Name -like "*$UserToIgnore*")
 			{
 			    $ThisProfileShouldBeDeleted=$False
 			}
 
 		}
 
 		$CurrentProfileName=$LocalProfileFolder.FullName
 
 		if ($ThisProfileShouldBeDeleted -eq $True)
 		{
 		    $ProfileToLookFor=$BaseProfilePath + $LocalProfileFolder.Name
 		    $ProfileToClean=$ProfileToLookFor -replace "\\","\\"
 		    $WMIQuery = $null
 		    $WMIQuery=Get-WmiObject Win32_UserProfile  -computer '.' -filter "localpath='$ProfileToClean'"
 
 			if ($WMIQuery -ne $null) {
     		        $TheProfileToDrop=$WMIQuery | Select-Object LocalPath
 	                $WMIProfileToDrop=$TheProfileToDrop.LocalPath
 	    	        Write-Output "Deleting $WMIProfileToDrop through WMI."
 		            $WMIQuery.Delete()
 			}
             else {
                 cmd /c rd /s /q "$CurrentProfileName"
                 Write-Output "No WMI-profile found, $CurrentProfileName folder was deleted though."
             }
         
     		Write-Output "$CurrentProfileName has been deleted."
 		}
 
 		else {
 	    	Write-Output "$CurrentProfileName was skipped."
 		}
 	}
 }
`

