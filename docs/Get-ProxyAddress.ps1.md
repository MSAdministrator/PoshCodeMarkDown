---
Author: paul brice
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6520
Published Date: 2016-09-15t00
Archived Date: 2016-10-29t06
---

# get-proxyaddress - 

## Description

this script enables you too search ad for smtp addresses that are possibly in use, using the quest powershell pssnapin and searching the “proxyaddress” attribute of objects. the output details either that the smtp address is not found in ad or you get details of the object that owns the smtp address.

## Comments



## Usage

ps c

## TODO



## function

`get-proxyaddresses`

## Code

`#
 #
 Param (
     [Parameter(Mandatory=$true,
         Position=0,
         ValueFromPipeline=$true,
         HelpMessage="Enter SMTP address to search for in Active-Directory."
     )]
     [string]$objSMTP
 	)
 Function Get-ProxyAddresses ([string]$Address){
 $objAD = $null
 $objAD = Get-QADObject -LdapFilter "(proxyAddresses=*$Address*)" -IncludeAllProperties -SizeLimit 0 -ErrorAction SilentlyContinue
 Write-Output $objAD
 Add-PSSnapin -Name Quest.ActiveRoles.ADManagement -ErrorAction SilentlyContinue
 $Results = $null
 $Results = Get-ProxyAddresses -Address $objSMTP | Select-Object Name,DisplayName,ObjectClass,Email,AccountisDisabled,AccountisLockedOut,MailNickName,LegacyExchangeDN -ErrorAction SilentlyContinue
 IF($Results -eq $null){
 Write-Host ""
 Write-Host "No Object Found with .attribute[proxyAddress] containing $objSMTP."}
 Else{$Results | Format-List *}
`

