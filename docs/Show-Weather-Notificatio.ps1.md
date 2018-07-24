---
Author: mosser lee
Publisher: 
Copyright: 
Email: 
Version: 273.15
Encoding: utf-8
License: cc0
PoshCode ID: 4634
Published Date: 2013-11-23t08
Archived Date: 2013-11-25t06
---

# show weather notificatio - 

## Description

show weather information in the lower right corner of the desktop.

## Comments



## Usage



## TODO



## function

`convertto-icon`

## Code

`#
 #
 <#
 #>
 
 
 function ConvertTo-Icon( [byte[]]$pngBuffer )
 {
     $pngStream=$null
     [void]($pngStream =New-Object 'io.memorystream' $pngBuffer,$true)
     $img=[System.Drawing.Image]::FromStream($pngStream)
     $icon=[System.Drawing.Icon]::FromHandle($img.GetHicon())
     return $icon
 }
 
 
 function Show-NotifyIcon([System.Drawing.Icon]$icon,[string]$title,[string]$message)
 {
     $balloon = New-Object System.Windows.Forms.NotifyIcon
     if($icon) 
     { 
         $balloon.Icon = $icon 
     }
     else 
     {
         $psProcPath =  Get-Process -id $pid | Select-Object -ExpandProperty Path
         $balloon.Icon = [System.Drawing.Icon]::ExtractAssociatedIcon( $psProcPath )
     }
     $balloon.BalloonTipIcon = 'Info'
     $balloon.BalloonTipText = $message
     $balloon.BalloonTipTitle = $title
     $balloon.Visible = $true
     $balloon.ShowBalloonTip(10000)
     sleep -Seconds 3
     $balloon.Dispose()
 }
 
 
 function Show-WeatherNotification([string]$cityName="Shanghai")
 {
     $weatherRequest =Invoke-WebRequest ( "$weatherProvider/data/2.5/weather?q={0}" -f $cityName )
     $weatherInfo = $weatherRequest | ConvertFrom-Json
 
     $message=""
 
     if($weatherInfo.cod -eq '200')
     {
         $temp_const = 273.15
          
         $city = $weatherInfo.name
         $des = $weatherInfo.weather.description
         $max_temp = [math]::Round( $weatherInfo.main.temp_max - $temp_const,1)
         $min_temp = [math]::Round( $weatherInfo.main.temp_min - $temp_const,1)
         $message="{0}: {1}`nTemperature({2}-{3})" -f $city,$des,$max_temp,$min_temp
     }
 
     $iconName = $weatherInfo.weather.icon
     [byte[]]$pngBuffer = (Invoke-WebRequest -Uri "$weatherProvider/img/w/$iconName.png" ).Content
     [System.Drawing.Icon]$icon = ConvertTo-Icon -pngBuffer $pngBuffer
     Show-NotifyIcon -icon $icon -title ¡®Weather info from PStips.net¡¯ -message $message
 }
 Add-Type -AssemblyName 'System.Drawing'
 Add-Type -AssemblyName 'System.Windows.Forms'
 $weatherProvider="http://api.openweathermap.org"
 
 Show-WeatherNotification -cityName Shanghai
`

