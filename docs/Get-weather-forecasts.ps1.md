---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 12.5
Encoding: ascii
License: cc0
PoshCode ID: 6590
Published Date: 2017-10-24t21
Archived Date: 2017-05-22t03
---

# get weather forecasts - 

## Description

this cmdlet retrieves weather forcasts from smhi (swedish meteorological and hydrological institute) through their api. (see

## Comments



## Usage



## TODO



## function

`get-smhiweatherdata`

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 function Get-SMHIWeatherData
 {
     [cmdletbinding()]
     param(
           [Parameter(Mandatory=$True, ValueFromPipelineByPropertyName=$true)]
           $Latitude,
           [Parameter(Mandatory=$True, ValueFromPipelineByPropertyName=$true)]
           $Longitude)
 
     BEGIN {
         $Data = $null
         $DailyForecast = $null
         $PreviousForecast = $null
     }
 
     PROCESS {
 
         $RoundedLong = [System.Math]::Round($Longitude,6)
         $RoundedLat = [System.Math]::Round($Latitude,6)
 
         $URL = "http://opendata-download-metfcst.smhi.se/api/category/pmp1.5g/version/2/geotype/point/lon/$RoundedLong/lat/$RoundedLat/data.json"
 
         try {
             $Data = Invoke-RestMethod -Uri $URL -Method Get -ErrorAction Stop
         }
         catch {
             Write-Error "Failed to get weather data from SMHI. The error was: $($Error[0])."
         }
 
         foreach ($DailyForecast in $Data.timeseries) {
             $ConvertedObj = $DailyForecast.parameters | % { @{ "$($_.name)" = "$($_.values)" } }
 
             $PrecipitationCategory = switch ($ConvertedObj.pcat)
             {
                       0 { "None" }
                       1 { "Snow" }
                       2 { "Snow and rain" }
                       3 { "Rain" }
                       4 { "Drizzle" }
                       5 { "Freezing rain" }
                       6 { "Freezing drizzle" }
                 default { "Unknown" }
             }
 
             $returnObject = New-Object System.Object
 
             if ($PreviousForecast -eq $null) {
                 $returnObject | Add-Member -Type NoteProperty -Name ForecastStartDate -Value $(Get-Date $Data.referenceTime -Format "yyyy-MM-dd HH:mm:ss")
             }
             else {
                 $returnObject | Add-Member -Type NoteProperty -Name ForecastStartDate -Value $(Get-Date $PreviousForecast -Format "yyyy-MM-dd HH:mm:ss")
             }
 
             $returnObject | Add-Member -Type NoteProperty -Name ForecastEndDate -Value $(Get-Date $DailyForecast.validTime -Format "yyyy-MM-dd HH:mm:ss")
             $returnObject | Add-Member -Type NoteProperty -Name Pressure -Value $ConvertedObj.msl
             $returnObject | Add-Member -Type NoteProperty -Name Temperature -Value $ConvertedObj.t
             $returnObject | Add-Member -Type NoteProperty -Name Visibility -Value $ConvertedObj.vis
             $returnObject | Add-Member -Type NoteProperty -Name WindDirection -Value $ConvertedObj.wd
             $returnObject | Add-Member -Type NoteProperty -Name WindVelocity -Value $ConvertedObj.ws
             $returnObject | Add-Member -Type NoteProperty -Name RelativeHumidity -Value $ConvertedObj.r
             $returnObject | Add-Member -Type NoteProperty -Name ThunderstormProbability -Value $ConvertedObj.tstm
             $returnObject | Add-Member -Type NoteProperty -Name TotalCloudCover -Value $(([int] $ConvertedObj.tcc)*12.5)
             $returnObject | Add-Member -Type NoteProperty -Name LowCloudCover -Value $(([int] $ConvertedObj.lcc)*12.5)
             $returnObject | Add-Member -Type NoteProperty -Name MediumCloudCover -Value $(([int] $ConvertedObj.mcc)*12.5)
             $returnObject | Add-Member -Type NoteProperty -Name HighCloudCover -Value $(([int] $ConvertedObj.hcc)*12.5)
             $returnObject | Add-Member -Type NoteProperty -Name WindGust -Value $ConvertedObj.gust
             $returnObject | Add-Member -Type NoteProperty -Name PrecipitationIntensityTotal -Value $ConvertedObj.pit
             $returnObject | Add-Member -Type NoteProperty -Name PrecipitationIntensitySnow -Value $ConvertedObj.pis
             $returnObject | Add-Member -Type NoteProperty -Name PrecipitationCategory -Value $PrecipitationCategory
 
             Write-Output $returnObject
 
             $PreviousForecast = $DailyForecast.validTime
         }
     }
 
     END { }
 }
`

