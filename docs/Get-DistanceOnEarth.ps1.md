---
Author: callias
Publisher: 
Copyright: 
Email: 
Version: 6378.1
Encoding: ascii
License: cc0
PoshCode ID: 2591
Published Date: 2011-03-29t06
Archived Date: 2016-05-28t20
---

# get-distanceonearth - 

## Description

uses the haversine equation (you know you love spherical geometry) to calculate distance between two points on the earth.

## Comments



## Usage



## TODO



## function

`get-distanceonearth`

## Code

`#
 #
 function Get-DistanceOnEarth {
 <#
 .SYNOPSIS
   Calculates distance between points on the Earth.
 .DESCRIPTION
   Implementation of the Haversine equation to calculate distance over the surface of a sphere.
 .INPUTTYPE
   Pipeline object with the following parameters: LATITUDE1 LONGITUDE1 LATITUDE2 LONGITUDE2
 #>
     Begin
     {
     }
     
     Process
     {
         $lat1 = $_.LATITUDE1
         $lon1 = $_.LONGITUDE1
         $lat2 = $_.LATITUDE2
         $lon2 = $_.LONGITUDE2
         $dLat = ($lat2 - $lat1) * $toRad
         $dLon = ($lon2 - $lon1) * $toRad
         $a = ( ( ([Math]::Sin($dLat/2)) * ([Math]::Sin($dLat/2)) ) + ([Math]::Cos($lat1 * $toRad) * [Math]::Cos($lat2 * $toRad) * ( ([Math]::Sin($dLon/2)) * ([Math]::Sin($dLon/2)) )))
         $c = 2 * [Math]::Atan2([Math]::Sqrt($a), [Math]::Sqrt(1-$a))
         $d = $R * $c
         $d = [Math]::Round($d,3)
         $_ | Add-Member -MemberType NoteProperty -Name Distance -Value $d
         Write-Output $_
     }
 }
`

