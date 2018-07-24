---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 250.0
Encoding: utf-8
License: cc0
PoshCode ID: 6377
Published Date: 2016-06-10t15
Archived Date: 2016-10-18t13
---

# showui weather widget - 

## Description

this is a simple weather forecast widget that shows the current temperature and forecast. don’t forget to change the $woeid to the right one for your location.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 New-UIWidget -AsJob { 
     Grid {
         Rectangle -RadiusX 10 -RadiusY 10 -StrokeThickness 0 -Width 170 -Height 80 -HorizontalAlignment Left -VerticalAlignment Top -Margin "60,40,0,0" -Fill { 
             LinearGradientBrush -Start "0.5,0" -End "0.5,1" -Gradient {
             }
         }
         Image -Name Image -Stretch Uniform -Width 250.0 -Height 180.0 -Source "http://l.yimg.com/a/i/us/nws/weather/gr/31d.png"
         TextBlock -Name Temp -Text "99°" -FontSize 80 -Foreground White -Margin "130,0,0,0" -Effect { DropShadowEffect -Color Black -Shadow 0 -Blur 8 }
         TextBlock -Name Forecast -Text "Forecast" -FontSize 12 -Foreground White -Margin "120,95,0,0"
     }
 } -Refresh "00:10" {
     $woEID = 14586
     $channel = ([xml](New-Object Net.WebClient).DownloadString("http://weather.yahooapis.com/forecastrss?p=$woEID")).rss.channel
     $h = ([int](Get-Date -f hh))
     
     if($h -gt ([DateTime]$channel.astronomy.sunrise).Hour -and $h -lt ([DateTime]$channel.astronomy.sunset).Hour) {
         $dayOrNight = 'd'
     } else {
         $dayOrNight = 'n'
     }
     $source = "http`://l.yimg.com/a/i/us/nws/weather/gr/{0}{1}.png" -f $channel.item.condition.code, $dayOrNight
     
     $Image.Source  = $source
     $Temp.Text     = $channel.item.condition.temp + [char]176
     $Forecast.Text = "High: {0}{2} Low: {1}{2}" -f $channel.item.forecast[0].high, $channel.item.forecast[0].low, [char]176
 }
`

