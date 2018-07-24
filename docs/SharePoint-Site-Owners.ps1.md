---
Author: lucas araujo
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4347
Published Date: 2015-07-31t21
Archived Date: 2015-05-16t08
---

# sharepoint site owners - 

## Description

list all the members of the “associatedownergroup” of each site (including root site) of each site collection of each web application in the farm.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Get-SPWebApplication | Get-SPSite -Limit ALL | 
 ForEach-Object {
     $content = "";
     $rootSite = New-Object Microsoft.SharePoint.SPSite($_.Url)
     $subSites = $rootSite.AllWebs;
     
     if($subSites.Count -le 0)
     {
 	@@ This occurs when a Site Collection does not contains any subsite (not even the root site)
         Write-Host "The Site Collection"  $siteUrl  "does not contains any site."
     }
     else
     {
         foreach($subsite in $subSites) 
         {
             $siteOwners = $subsite.AssociatedOwnerGroup
             if($siteOwners)
             {
                 foreach ($owner in $subsite.Users) 
                 {
                     if($subsite.ParentWeb)
                     {
                         $content += "$($subsite.ParentWeb.Url),$($owner.Name),Site Owner`n"
                     }
                     else
                     {
                         $content += "$($subsite.Url),$($owner.Name),Site Owner`n"
                     }
                 }
             }
             else
             {
                 $content += "Could not find an AssociatedGroupOwner in the site with the Url: $($subsite.Url) `n"
             }  
             
             $subsite.Dispose()
             $rootSite.Dispose()
         }
         @@ Set the patch to the CSV files
         $FilePath = "C:\Owners\" + $_.Url.Replace("http://","").Replace("/","-").Replace(":","-") + ".csv";
 	out-file -filepath $FilePath -inputobject $content
     }
 }
`

