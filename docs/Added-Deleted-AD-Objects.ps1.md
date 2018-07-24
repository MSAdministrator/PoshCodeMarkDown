---
Author: joel de la torre
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 142
Published Date: 2008-02-12t14
Archived Date: 2016-03-20t09
---

# added/deleted ad objects - 

## Description

i earlier converted a vbscript written by chrissy lemaire to powershell but later wanted to add in some code to include active directory objects that have been deleted.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 begin {
 
 $strSMTPServer = "xx.xx.xx.xx";
 $strEmailFrom = "AD_Admin@yourdomain.com";
 $strEmailTo = "admin@yourdomain.com";
 $borders = "=" * 25;
 [int]$days = -60
 
 function TombStonedObjects {
 
 	$ds = New-Object System.DirectoryServices.DirectorySearcher
 	$ds.Tombstone = $TRUE
 	$ds.Filter = "isDeleted=TRUE"
 
 	$DSResults=$DS.FindAll() | select path
 
 	$r=[regex]"(?<=CN=).+(?=\\)"
 	$DSR2=$DSResults | % { $r.Matches($_);$script:delCount++}
 	foreach ($DSobject in $DSR2) { $delMessage += "Deleted object: " + $DSobject.value.trim() + "`n" }
 	
 	$delMessage
 	
 	}
 
 
 function AddedComputersAndUsers {
 $ADObjects=Get-QADObject | ? {$_.type -match ("computer|user")} | ? {$_.whencreated -gt ((get-date).addDays($days))}
 
   if ($ADObjects) {
 	foreach ($ADObject in $ADObjects) {
 		switch ($ADObject.Type) {
 			'user'	{
 				$usrCount ++;
 				$usrMessage += "Display Name: " + $ADobject.displayname + "`n";
 				$usrMessage += "SAMAccountName: " + $ADObject.get_LogonName() + "`n";
 				$usrMessage += "Container: " + $ADObject.parentcontainer + "`n";
 				$usrMessage += "When Created: " + $ADObject.whencreated + "`n";
 				$usrMessage += "Principal Name: " + $ADObject.userPrincipalName + "`n";
 				$usrMessage += "Groups: `n";
 					$groups=$adobject.MemberOf
 					foreach ($group in $groups) { $usrMessage += "$group `n"}
 				$usrMessage += "`n";
 				}				
 			'computer' {
 				$computerCount ++;
 				$compMessage += "DNS HostName: " + $ADObject.dnsname + "`n";
 				$compMessage += "OperatingSystem: " + $ADObject.osName + "`n";
 				$compMessage += "OS Service Pack: " + $ADObject.osservicepack + "`n";
 				$compMessage += "Computer Role: " + $ADObject.computerrole + "`n";
 				$compMessage += "When Created: " + $ADObject.whencreated + "`n";
 				$compMessage += "Container: " + $ADObject.parentcontainer + "`n";
 				$compMessage += "`n";
 				}
 			}
 		}
 	
 	$deletedobjects = TombStonedObjects
 	
 	$script:emailMessage = "AD User/Computer Objects created in the last " + [math]::abs($days) + " day(s).`n";
 	if ($usrMessage) {$script:emailMessage += "$borders Users $borders`n" + $usrMessage;}
 	if ($compMessage) {$script:emailMessage += "$borders Computers $borders`n" + $compMessage;}
 	if ($deletedobjects) {$script:emailMessage += "$borders Deleted Objects for the last 60 days $borders `n" + $deletedobjects;}
 	$script:emailSubject = "Users Added: " + $usrCount + ". Computers Added: " + $computerCount + ".  Objects Deleted: " + $script:delCount + ".";
 	
 	}
 	
 	else {
 	$deletedobjects = TombStonedObjects
 	$script:emailSubject = "Users Added: " + $usrCount + ". Computers Added: " + $computerCount + ".  Objects Deleted: " + $script:delCount + ".";
 	$script:emailMessage = "No Users or Computers have been added in the last " + [math]::abs($days) + " day(s). `n";
 	if ($deletedobjects) {$script:emailMessage += "$borders Deleted Objects for the last 60 days $borders `n" + $deletedobjects;}
 		}
 	}
 }
 
 
 process {
 
 
 AddedComputersAndUsers
 Send-SmtpMail -Subject $script:emailSubject -To $strEmailTo -From $strEmailFrom -SmtpHost $strSMTPServer -Body $script:emailMessage;
 
 }
`

