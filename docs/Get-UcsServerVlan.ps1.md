---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 3813
Published Date: 2013-12-04t08
Archived Date: 2016-07-04t13
---

# get-ucsservervlan - 

## Description

uses the cisco ucs powertool (http

## Comments



## Usage



## TODO



## function

`get-ucsservervlan`

## Code

`#
 #
 function Get-UcsServerVlan {
     Get-UcsServiceProfile | Foreach-Object {
         $sp = $_
         $sp | Get-UcsVnic | Foreach-Object {
             $vn = $_
             $vn | Get-UcsVnicInterface | Foreach-Object {
                 $output = New-Object psobject �property @{
                     Server = $sp.Name
                     Vnic = $vn.name
                     Vlan = $_.name
                 }
                 Write-Output $output
             }
         }
     }
 }
`

