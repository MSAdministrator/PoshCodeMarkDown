---
Author: karl prosser
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2571
Published Date: 2012-03-19t14
Archived Date: 2012-11-30t21
---

# where-property - 

## Description

different examples of how you can access properties with a custom where function

## Comments



## Usage



## TODO



## function

`where-property`

## Code

`#
 #
 function where-property([string] $PropertyName,[string]$SubProperty , $is,$isnot,$contains,$in,$SelectProperty)  
 {
 
  process {
     $useprop = $SelectProperty
   Function _outobj {
     if ($useprop ) 
        {
         , $_.$useprop
        }
     else 
        {
         , $_
         }
     }
     if ($is) 
         {
            if ($_.$Propertyname -eq $is) { _outobj}
         }
     elseif ($isnot) 
         {  
             if ($_.$Propertyname -ne $is) { _outobj} 
         }
     elseif($contains)
         { 
                 if ($subproperty)
                 {
                     foreach ($prop in  $_.$propertyname )
                     {
                       if ($prop)
                        {                         
                          $foundmatch = $false
                          $subpropertyvalue = $prop.$subproperty
                          if ($subpropertyvalue -contains $contains ) { $foundmatch = $true }
                          if ($foundmatch) { _outobj } 
                        }
                     }
                 }
                 else 
                 {  
                     if ($_.$Propertyname -contains $contains) { _outobj}            
                 }
             
         }
     elseif($in)
         { 
              if ($subproperty)
                 {
                     foreach ($prop in  $_.$propertyname )
                     {
                       if ($prop)
                        {                         
                          $foundmatch = $false        
                          #{  "RpcSs","AppID" -contains $_.servicesdependedon.name }                 
                          if ($in -contains $prop.$subproperty ) 
                             {
                              $foundmatch = $true 
                              }
                          if ($foundmatch) { _outobj } 
                        }
                     }
                 }
                 else 
                 {  
                     if ($in -contains $_.$Propertyname ) { _outobj}            
                 }
         }    
      
     
  }
 }
 set-alias ?. where-property
 set-alias and. where-property
 set-alias and-property where-property
 
 
 
 gps | where-property processname -is svchost
 gps | where-property processname -isnot svchost
 gps | where-property processname -in iexplore,chrome
 get-verb | where-property group -in common
 
 
  get-command | where-property parameters -subproperty keys  -contains Begin 
  get-command | where-property parameters -subproperty keys  -contains Name |  and-property commandtype -is alias
 
 get-service | where-property ServicesDependedOn -SubProperty name -in AppID,rpcss | and-property status -is running -SelectProperty Displayname
`

