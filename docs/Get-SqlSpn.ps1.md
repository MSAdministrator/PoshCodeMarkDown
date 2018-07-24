---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3234
Published Date: 2012-02-14t18
Archived Date: 2012-02-18t15
---

# get-sqlspn - 

## Description

gets mssqlsvc service principal names from active directory

## Comments



## Usage



## TODO



## script

`get-sqlspn`

## Code

`#
 #
 #######################
 <#
 .SYNOPSIS
 Gets MSQLSvc service principal names (spn) from Active Directory.
 .DESCRIPTION
 The Get-SqlSpn function gets SPNs for MSQLSvc services attached to account and computer objects
 .EXAMPLE
 Get-SqlSpn
 This command gets MSSQLSvc SPNs for the current domain
 .NOTES 
 Adapted from http://www.itadmintools.com/2011/08/list-spns-in-active-directory-using.html
 Version History 
 v1.0   - Chad Miller - Initial release 
 #>
 function Get-SqlSpn
 {
     $serviceType="MSSQLSvc"
     $filter = "(servicePrincipalName=$serviceType/*)"
     $domain = New-Object System.DirectoryServices.DirectoryEntry
     $searcher = New-Object System.DirectoryServices.DirectorySearcher
     $searcher.SearchRoot = $domain
     $searcher.PageSize = 1000
     $searcher.Filter = $filter
     $results = $searcher.FindAll()
 
     foreach ($result in $results) {
         $account = $result.GetDirectoryEntry()
         foreach ($spn in $account.servicePrincipalName.Value) {
             if($spn -match "^MSSQLSvc\/(?<computer>[^\.|^:]+)[^:]*(:{1}(?<port>\w+))?$") {
                 new-object psobject -property @{ComputerName=$matches.computer;Port=$matches.port;AccountName=$($account.Name);SPN=$spn} 
 
             } 
         }
     }
 
`

