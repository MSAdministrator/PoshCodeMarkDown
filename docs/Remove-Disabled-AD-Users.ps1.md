---
Author: drdrewl
Publisher: 
Copyright: 
Email: 
Version: 400.00
Encoding: ascii
License: cc0
PoshCode ID: 6129
Published Date: 2016-12-04t10
Archived Date: 2016-07-13t12
---

# remove disabled ad users - 

## Description

this script is a simple one that is meant to be scheduled on a periodic basis (we do it weekly). it looks inthe ou where we put our disabled ad users and removes users that have not logged in (inactive) for 400 days. this allows us to keep terminated employees disabled users for over a year for auditing purposes, but automatically cleans them out once the annual scope has passed.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 import-module ActiveDIrectory
 search-adaccount -searchbase "ou=UserObjectsPendingDeletion,DC=mydomain,DC=com" -Accountinactive -Timespan 400.00:00:00 | where {$_.objectclass -eq 'user'} |  remove-aduser -confirm:$false
`

