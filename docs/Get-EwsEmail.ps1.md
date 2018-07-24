---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 2978
Published Date: 2012-09-28t20
Archived Date: 2016-06-04t02
---

# get-ewsemail - 

## Description

this is a sample script to read emails from an inbox using exchange web services. the code is a basic port of the c# found here

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $ewsPath = "C:\Program Files\Microsoft\Exchange\Web Services\1.1\Microsoft.Exchange.WebServices.dll"
 Add-Type -Path $ewsPath
 
 $ews = New-Object Microsoft.Exchange.WebServices.Data.ExchangeService -ArgumentList "Exchange2007_SP1"
 $cred = (Get-Credential).GetNetworkCredential()
 $ews.Credentials = New-Object System.Net.NetworkCredential -ArgumentList $cred.UserName, $cred.Password, $cred.Domain
 $ews.AutodiscoverUrl( ( Read-Host "Enter mailbox (email address)" ) )
 $results = $ews.FindItems(
 	"Inbox",
 	( New-Object Microsoft.Exchange.WebServices.Data.ItemView -ArgumentList 10 )
 )
 $results.Items | ForEach-Object { $_.Subject }
`

