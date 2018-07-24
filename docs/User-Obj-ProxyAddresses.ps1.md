---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1026
Published Date: 
Archived Date: 2009-09-15t18
---

# user obj proxyaddresses - 

## Description

a code sample to gather all assigned proxy addresses of an active directory user object.

## Comments



## Usage



## TODO



## function

`get-proxy`

## Code

`#
 #
 Function Get-Proxy()
 {
 Process
 {
 $objDomain = New-Object System.DirectoryServices.DirectoryEntry
 $objSearcher = New-Object System.DirectoryServices.DirectorySearcher
 $objSearcher.SearchRoot = $objDomain
 $objSearcher.PageSize = 1000
 $objSearcher.Filter = "(cn=$_)"
 $objSearcher.SearchScope = "Subtree"
 $objUser = $objSearcher.FindOne()
 [String]$DN = $objUser.properties.distinguishedname
 $UserObj = [ADSI]"LDAP://$DN"
 [String]$displayname = $UserObj.displayname
 [String]$exchalias = $UserObj.mailnickname
 [Array]$exchproxy = $UserObj.proxyaddresses
 $displayname
 $exchalias
 ForEach($proxy In $exchproxy)
 {
 $proxy
 }
 }
 }
 [String]$UserCN = "bricep"
 $UserCN | Get-Proxy
`

