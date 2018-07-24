---
Author: brodobrey
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3628
Published Date: 2012-09-07t06
Archived Date: 2012-09-15t04
---

# createsite_tmp.ps1 - 

## Description

createsite_tmp.ps1

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
  $site = Get-SPSite http://spf/
 $web = $site.RootWeb
 
 write-host "template = $templates ; web = $web "
 
 New-SPWeb -name 'KoKA2' -url http://spf/koka2 -UseParentTopNav -AddToTopNav -Template  $templates 
 
  
 #" -UseParentTopNav -UniquePermissions
  
  #>
`

