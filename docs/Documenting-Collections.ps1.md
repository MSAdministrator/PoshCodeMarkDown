---
Author: brian skinner
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5617
Published Date: 2014-12-01t15
Archived Date: 2014-12-13t19
---

# documenting collections - 

## Description

small script to export a csv file containing a collections id, name and limiting collection id. this can be used as is to import into microsoft visio org chart wizard to produce a graphical representation of your collection structure.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###################################################################
 ###################################################################
 
 function DocumentCollectionTree
 {
     param($LimitCollectionID)
     $info=@()
     $subcollections = Get-WmiObject `
                         -ComputerName $siteserver `
                         -namespace root\sms\site_$sitecode `
                         -query "select * from SMS_Collection where LimitToCollectionID = '$LimitCollectionID'"
 
     if ($subcollections -ne $null)
     {
         foreach ($subcoll in $subcollections)
         {
             $object = New-Object -TypeName PSObject
             $object | Add-Member -MemberType NoteProperty  -Name CollectionID -Value $subcoll.collectionid
             $object | Add-Member -MemberType NoteProperty  -Name CollectionName -Value $subcoll.name
             $object | Add-Member -MemberType NoteProperty  -Name LimitingCollectionID -Value $LimitCollectionID
             $info += $object
             DocumentCollectionTree $subcoll.collectionid
         }
     }
     $info | export-csv -path "c:\temp\collections.csv" -NoTypeInformation -Append -Force
  }
 
 
 $collectionid = Read-Host "Enter the root collection id to search from" 
 $siteserver = "ServerName"
 $sitecode = "sitecode"
 
 DocumentCollectionTree $collectionid
`

