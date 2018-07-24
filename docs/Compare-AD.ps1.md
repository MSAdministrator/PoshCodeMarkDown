---
Author: dmitry sotnikov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 638
Published Date: 2008-10-14t21
Archived Date: 2016-11-12t21
---

# compare-ad - 

## Description

a set of functions (and sample code using them) to create

## Comments



## Usage



## TODO



## function

`new-snapshot`

## Code

`#
 #
 ###############################################################################
 #
 ###############################################################################
 
 if ((Get-PSSnapin "Quest.ActiveRoles.ADManagement" `
 			-ErrorAction SilentlyContinue) -eq $null) {
 	Add-PSSnapin "Quest.ActiveRoles.ADManagement"
 }
 
 
 ###############################################################################
 ###############################################################################
 
 function New-Snapshot {
     param($path, $properties)
     Get-QADUser -SizeLimit 0 -DontUseDefaultIncludedProperties `
 		-IncludedProperties $properties | 
 			Select $properties | Export-Clixml $path
 }
 
 function Compare-ActiveDirectory {
     param($path, $properties)
 	
     $old = Import-Clixml $path
     $new = Get-QADUser -SizeLimit 0 -DontUseDefaultIncludedProperties `
 		-IncludedProperties $properties | 
         	Select $properties
 
     $diff = Compare-Object $old $new -Property Name
     $created = , ($diff | where { $_.SideIndicator -eq "=>" })
     $deleted = , ($diff | where { $_.SideIndicator -eq "<=" })
     
     if ( $created.Count -gt 0 ) {
         "New accounts:"
         $created | Format-Table Name
     }
     
     if ( $deleted.Count -gt 0 ) {
         "Deleted accounts:"
         $deleted | Format-Table Name
     }
     
     $hash = @{}
     $new | ForEach-Object { $hash[$_.DistinguishedName] = $_ }
     
     "Modified objects:"
     foreach ( $snapshot in $old ) {
         $current = $hash[$snapshot.DistinguishedName]
         if ( $current -and 
             ($current.ModificationDate -ne $snapshot.ModificationDate )) {
 @"
 
 Object $($snapshot.distinguishedname)
 Modified at $($current.ModificationDate)
                 
 Property`tOld Value`tNew Value
 ========`t=========`t=========
 "@
 			foreach ($property in $properties) {
 				if ( ($property -ne "ModificationDate") -and 
 					($snapshot.$property -ne $current.$property )) {
 				"$property`t$($snapshot.$property)`t$($current.$property)"
 				}
 			}
 		}
     }
 }
 
 ###############################################################################
 ###############################################################################
 
 $Members_to_Compare = @( "Name", "DistinguishedName", "ModificationDate", 
     "AccountIsDisabled", "AccountIsLockedOut", "AccountName", 
     "CanonicalName", "City", "Company", "Department", "Description", 
     "DisplayName", "Email", "Fax",
     "FirstName", "HomeDirectory", "HomeDrive", "HomePhone", "Initials", 
     "LastName", "LdapDisplayName", "LogonName", "LogonScript", "Manager", 
     "MobilePhone", "Notes", "Office", "Pager", 
     "ParentContainerDN", "PasswordNeverExpires", 
     "PhoneNumber", "PostalCode", "PostOfficeBox", "ProfilePath", 
     "SamAccountName", "StateOrProvince", "StreetAddress", "Title", 
     "TsAllowLogon", "TsBrokenConnectionAction", "TsConnectClientDrives", 
     "TsConnectPrinterDrives", "TsDefaultToMainPrinter", "TsHomeDirectory", 
     "TsHomeDrive", "TsInitialProgram", "TsMaxConnectionTime", 
     "TsMaxDisconnectionTime", "TsMaxIdleTime", "TsProfilePath", 
     "TsReconnectionAction", "TsRemoteControl", "TsWorkDirectory", 
     "UserPrincipalName", "WebPage" )
 
 $SnapshotPath = "c:\snapshot.xml"
 
 ###############################################################################
 ###############################################################################
 
 New-Snapshot -Path $SnapshotPath -Properties $Members_to_Compare
 
 New-QADUser -Name "Lawrence Alford"  -ParentContainer mydomain.local/test
 Set-QADUser "Jennifer Clarke" -PhoneNumber "(249) 111-22-33"
 Set-QADUser "Ernest Cantrell" -City "San Francisco"
 Remove-QADObject "Andreas Bold"  -Force
 
 Compare-ActiveDirectory -Path $SnapshotPath -Properties $Members_to_Compare
`

