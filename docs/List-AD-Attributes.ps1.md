---
Author: bsonposh
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2755
Published Date: 2011-06-28t05
Archived Date: 2011-07-06t05
---

# list ad attributes - 

## Description

list active directory attributes from schema

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $forest = [DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest()
 $Schema = $forest.schema 
 $Properties = $Schema.FindAllProperties()
 foreach($property in $Properties)
 {
    "#################################"
    "Name:   {0}" -f $property.Name
    "Link:   {0}" -f $property.link
    "LinkID: {0}" -f $property.linkid
    if(!$?)
    {
         "Error: {0}" -f $error[0].message
    }
    "#################################"
 }
`

