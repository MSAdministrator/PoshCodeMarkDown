---
Author: alex scoble
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3068
Published Date: 2012-11-22t16
Archived Date: 2016-04-27t15
---

# user entitlement auditor - 

## Description

script used to perform user entitlement audits based on an xml report containing local groups, members of the local groups and systems that house the local groups. the script joins that data to data containing users and global groups grabbed directly from ad using the quest ad cmdlets and outputs the final report in csv. sorry that not everything is documented as i’d like it to be.

## Comments



## Usage



## TODO



## function

`is-validmember`

## Code

`#
 #
 $date = ( get-date ).ToString('yyyy-MM-dd-hh-mm')
 
 $inpath = "C:\InputDir\"
 
 $outpath = "C:\Archive\"
 
 $FileList = @(Get-Childitem $inpath)
 
 $ResultFile = [System.IO.Path]::GetTempFileName()
 
 $ADMembersOutfile = [System.IO.Path]::GetTempFileName()
 
 $ReportFile = "C:\EntitlementsAuditProject\$date-UserEntitlementAuditTripWireFINALresult.csv"
 
 $WantedLocals = @("Administrators","Users")
 
 $BadUsers = @("(null)", "") 
 
 $BadDomains = @("NT AUTHORITY","NT SERVICE") 
 
 Function Main {
 
 	Parse-XMLtoCSV $ResultFile
 	
 	JoinCSVs $ResultFile $ADMembersOutfile $ReportFile
 
 }
 
 Function Is-ValidMember([string] $membername) {
 
     $parts = $membername.Split("\\")
 
     $BadDomains -notcontains $parts[0] -and $BadUsers -notcontains $parts[$parts.Length - 1]
     
 }
 
 Function Is-ValidLocalGroup([string] $localgroupname) {
 
 	$WantedLocals -contains $localgroupname
 	
 }
 
 Function Parse-XMLtoCSV ([string] $outfile) {
 
 	$FileList | ForEach-Object {
 	
 		Foreach ($file in $FileList) {
 
 			[xml]$xml = Get-Content $inpath$file
 
 			$xml.ReportOutput.ReportBody.ReportSection | 
 
 			Foreach-Object { 
 
 				$system = $_.Name
 				
 				$_.ReportSection | 
 				
 				Foreach-Object {
 				
 					$group = $_.name
 					
 					$_.ReportSection.ReportSection.ReportSection | 
 					
 					Foreach-Object { 
 						
 						$_.String | 
 						
 						Foreach-Object {
 
 							
 								New-Object Object |
 								
 								Add-Member NoteProperty "System" $system -PassThru |
 								
 								Add-Member NoteProperty "LocalGroup" $group -PassThru |
 								
 								
 							}
 								
 						} 
 
 					} 
 						
 				}
 
 			} | Where-Object { Is-ValidMember $_.LocalMembers } | Export-Csv -NoTypeInformation $outfile
 
 		} 
 
 	Move-Item $inpath$file $outpath$file
 
 	}
 
 }
 
 Function GetADMembersofGroups ([string] $outfile2) {
 
 	Get-QADGroup -SearchRoot "DC=dsl,DC=dittdsh,DC=org" -GroupType "Security" `
 	-GroupScope "Global" | 
 	
 	ForEach-Object {
 		
 		$NTAccount = $_.NTAccountName
 		
 		$Domain = $_.Domain
 		
 		$GroupName =$_.Name
 		
 		$CatGroupName = [System.String]::Concat($Domain,$GroupName)
 		
 		Get-QADGroupMember -Identity $NTAccount -DontUseDefaultIncludedProperties -IncludedProperties `
 		"SamAccountName","Name"
 	
 	} | ForEach-Object {
 	
 		$curr = $_
 		
    	 	New-Object Object |
 
 		Add-Member NoteProperty "GlobalMembers" $CatGroupName -PassThru |
 
 		Add-Member NoteProperty "Username" $curr.Name -PassThru |
             
 		Add-Member NoteProperty "LogonID" $curr.SamAccountName -PassThru |
 
   	}  | Sort-Object {$_.GlobalMembers} | Export-Csv -NoTypeInformation $outfile2
 
 }
 
 Function JoinCSVs ([string] $infile2, [string] $infile3, [string] $outfile3) {
 
 	$csv1 = Import-Csv $infile2
 	
 	GetADMembersofGroups $infile3
 	
 	$csv2 = Import-Csv $infile3
 	
 	$csv1 | ForEach-Object { 
 		
 		$record = $_ 
 		
 		$matches = $csv2 | Where-Object {$_.GlobalMembers -eq $record.LocalMembers} 
 
 		if ($matches){ 
 		
 			$matches | ForEach-Object{ 
 	
 				New-Object Object |
 
 				Add-Member noteproperty Username $_.Username -PassThru |
 
 				Add-Member noteproperty LogonID $_.LogonID -PassThru |
 
 				Add-Member noteproperty Members $record.LocalMembers -PassThru |
 
 				Add-Member noteproperty Localgroup $record.Localgroup -PassThru |
 
 				Add-Member noteproperty System $record.System -PassThru |
 			
 			} 
 		
 		} 
 
 		else { 
 
 			New-Object Object | 
 
 			Add-Member noteproperty Username $null -PassThru |
 
 			Add-Member noteproperty LogonID $null -PassThru |
 			
 			Add-Member noteproperty Members $_.LocalMembers -PassThru |
 			
 			Add-Member noteproperty Localgroup $_.Localgroup -PassThru | 
 
 			Add-Member noteproperty System $_.System -PassThru | 
 
 		} 
 
 	} | Export-Csv -NoTypeInformation $outfile3 
 	
 	Remove-Item -Force $infile2
 	
 	Remove-Item -Force $infile3
 
 }
 
 . Main
`

