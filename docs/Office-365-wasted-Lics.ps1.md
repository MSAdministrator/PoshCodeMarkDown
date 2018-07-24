---
Author: adrianwoodrup
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3940
Published Date: 2015-02-12t14
Archived Date: 2015-04-14t07
---

# office 365 wasted lics - 

## Description

this script will create a csv file with all the users that are disabled in ad but still have licenses in office 365, it will then email the script out and perform a clean up.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 $AD = 'C:\temp\ad.csv'
 $o365 = 'C:\temp\o365.csv'
 $duplicates = 'C:\temp\Duplicate Office 365 users.csv'
 $log = 'C:\temp\Duplicate Office 365 users.log'
 $PSEmailServer = 'MAILSERVER'
 $EmailTo = 'TO-EMAILADDRESS'
 $EmailFrom = 'FROM-EMAILADDRESS'
 
 Import-Module ActiveDirectory 
 Import-Module MSOnline
 
 $password = ConvertTo-SecureString 'PASSWORD' -AsPlainText -Force
 $LiveCred = New-Object System.Management.Automation.PSCredential ("O365-USERNAME", $password)
 New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri https://ps.outlook.com/powershell/ -Credential $LiveCred -Authentication Basic -AllowRedirection
 
 Connect-MsolService -Credential $Livecred
 
 Search-ADAccount -AccountDisabled | Select-Object UserPrincipalName | where-object {$_.UserPrincipalName -like "*@*" }  | Export-Csv $AD
 Get-MsolUser -all | Where-Object { $_.isLicensed -eq "TRUE"} | Select-Object UserPrincipalName | Export-Csv $o365
 $Import = Import-Csv $o365
 $Import | Out-File -Append $AD -encoding ASCII
 
 $content = Get-Content $AD
 $content | Foreach {$_.TrimEnd()} | Set-Content $AD
 
 $RMVQuotes = Get-Content $AD 
 $RMVQuotes -replace('"','') | Sort-Object | Out-File $AD -Force
 Get-Content $AD | Group-Object -NoElement | Where-Object {$_.Count -gt 1} | Select-Object Count, Name | Export-Csv $duplicates
 
 get-pssession | where-object {$_.ComputerName -Like "*.outlook.com"} | remove-pssession
 
 copy-item $duplicates $log
 
 $DupLic = Import-Csv $duplicates | Measure-Object | Select-Object -Expand count
 $subject = "There are " + $DupLic + " office 365 Licenses being used by disabled users"
 if ($DupLic -gt 3){
 	Send-MailMessage -To $EmailTo -From $EamilFrom -Subject $Subject -attachments $log -Body "There are O365 Licenses being used by disabled users. Please remove theses licenses from Office 365 as each one costs. The attached is a list of all users that have an office 365 license assigned to them but are disabled in AD. Confirm the account and email is not in use and the license can be deleted. "
 }
 
 Remove-Item $AD
 Remove-Item $o365
 Remove-Item $Duplicates
 Remove-Item $Log
`

