---
Author: ant b 2012
Publisher: 
Copyright: 
Email: 
Version: 1.3
Encoding: ascii
License: cc0
PoshCode ID: 3351
Published Date: 2012-04-12t08
Archived Date: 2016-06-02t02
---

# mailchimp - 

## Description

a posh script to batch feed email address, firstname and lastname to the mailchimp api (mcapi)

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 
 if ( (Get-Module -Name ActiveDirectory -ErrorAction SilentlyContinue) -eq $null )
 {
     Import-Module ActiveDirectory
 }
 
 $ie = new-object -com "InternetExplorer.Application"
 
 $apikey = "<Your API Key>"
 $listid = "<Your list ID>"
 $URL = "https://us4.api.mailchimp.com/1.3/?output=xml&method=listBatchSubscribe&apikey=$apikey"
 $URLOpts = "&id=$($listid)&double_optin=False&Update_existing=True"
 $Header = "surname","givenanme","mail"
 
 $DGMembers = Get-AdGroupMember -Identity "<Full path to AD Group>" | ForEach {Get-ADUser $_.samaccountname -Properties mail} | Select GivenName, Surname, mail
 
 if (test-path "submitted.csv") {Clear-Content submitted.csv}
 $DGMembers | select mail | export-csv submitted.csv
 
 if (test-path "submitted.csv") {$list = Import-CSV -delimiter "," -Path submitted.csv -Header $Header}
 $Removals = compare-object $DGMembers $list | Where {$_.SideIndicator -eq "=>"}
 
 ForEach ($ObjItem in $DGMembers)
 {
 $batch = "&batch[0][EMAIL]=$($ObjItem.mail)&batch[0][FNAME]=$($ObjItem.givenname)&batch[0][LNAME]=$($ObjItem.surname)"
 $MailOpts = "&batch[0][EMAIL_TYPE]html"
 $FURL = $URL + $batch + $MailOpts + $URLOpts
 Write-Host $FURL
 $ie.navigate($FURL)
 Start-Sleep -MilliSeconds 300
 }
`

