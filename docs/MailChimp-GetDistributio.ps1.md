---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.3
Encoding: ascii
License: cc0
PoshCode ID: 3479
Published Date: 2012-06-25t09
Archived Date: 2015-12-06t18
---

# mailchimp getdistributio - 

## Description

a quick little script to get members of a distribution group to sync with mailchimp instead of using active directory.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 
 $ie = new-object -com "InternetExplorer.Application"
 
 $apikey = "api-key-from-mailchimp"
 $listid = "list-id-from-mailchimp"
 
 $URL = "https://us5.api.mailchimp.com/1.3/?output=xml&method=listBatchSubscribe&apikey=$apikey"
 $URLOpts = "&id=$($listid)&double_optin=False&Update_existing=True"
 $Header = "FirstName","Lastname","PrimarySmtpAddress"
 $GroupName = "distribution-group-name"
 
 $DGMembers = Get-DistributionGroupMember $GroupName | select FirstName, Lastname, PrimarySmtpAddress
 
 if (test-path "C:\submitted.csv") {
 	Clear-Content submitted.csv
 }
 $DGMembers | select PrimarySmtpAddress | export-csv C:\submitted.csv
 
 if (test-path "C:\submitted.csv") {
 	$list = Import-CSV -delimiter "," -Path C:\submitted.csv -Header $Header
 }
 $Removals = compare-object $DGMembers $list | Where {$_.SideIndicator -eq "=>"}
 
 ForEach ($ObjItem in $DGMembers)
 {
 	$batch = "&batch[0][EMAIL]=$($ObjItem.PrimarySmtpAddress)&batch[0][FNAME]=$($ObjItem.FirstName)&batch[0][LNAME]=$($ObjItem.Lastname)"
 	$MailOpts = "&batch[0][EMAIL_TYPE]html"
 	$FURL = $URL + $batch + $MailOpts + $URLOpts
 	Write-Host $FURL
 	$ie.navigate($FURL)
 	Start-Sleep -MilliSeconds 300
 }
`

