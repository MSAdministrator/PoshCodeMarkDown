---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1490
Published Date: 2009-11-25t22
Archived Date: 2014-12-03t03
---

# find-geocode - 

## Description

a quick demonstration of using the mappoint (aka bing maps?) web services to get addresses from lat,long pairs and vice-versa. requires you to sign up for (at least) the free developer account in order to use the api.

## Comments



## Usage



## TODO



## function

`find-reversegeocode`

## Code

`#
 #
 $mappoint = New-WebServiceProxy http://staging.mappoint.net/standard-30/mappoint.wsdl -Namespace MapPoint
 $FindService = new-object MapPoint.FindServiceSoap
 $FindService.Credentials = Get-Credential 
 
 function Find-ReverseGeoCode( [double]$Latitude, [double]$longitude, [switch]$force  ) 
 {
    $Coords = new-object MapPoint.LatLong
    $Coords.Latitude = $Latitude
    $Coords.Longitude = $longitude
    $Locations = $FindService.GetLocationInfo( $Coords, "MapPoint.NA", $null)
 
    if($force) {
       return $Locations
    } elseif($Locations[0].Address) {
    } else {
    }
 }
 
 function Find-GeoCode( $address, $country = "US" ) 
 {
    $spec = new-object MapPoint.FindAddressSpecification
    $spec.DataSourceName = "MapPoint.NA"
    $spec.InputAddress = $FindService.ParseAddress( $address,  $country )
    
    $Found = $FindService.FindAddress( $spec )
    if($Found.NumberFound) {
       $found.Results | Select -expand FoundLocation | Select -Expand LatLong
    }
 }
 
 Set-Alias Find-Address Find-ReverseGeoCode
 Set-Alias Find-Coordinates Find-GeoCode
`

