---
Author: brodobrey
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3630
Published Date: 2012-09-07t06
Archived Date: 2012-09-15t04
---

# createsitefromtemplate - 

## Description

createsitefromtemplate.ps1

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $templateID = "{055CF2A7-43A8-48E1-95CB-19DC393F0215}"
 
 $site= new-Object Microsoft.SharePoint.SPSite($url ) 
 $loc= [System.Int32]::Parse(1049) 
 
 $templates= $site.GetWebTemplates($loc) 
 
 
 foreach ($child in $templates){    write-host $child.Name "  " $child.Title} 
 $site.Dispose() 
 
 
 $web = New-SPWeb -Url http://spf/$targeturl -Name "$namesite" -UseParentTopNav -AddToTopNav -UniquePermissions
 
 
 
 #-template argument. Then you can apply the custom template 
 $web.ApplyWebTemplate($templateID)
`

