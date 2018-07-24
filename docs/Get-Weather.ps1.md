---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.1
Encoding: utf-8
License: cc0
PoshCode ID: 821
Published Date: 2010-01-23t13
Archived Date: 2017-01-05t17
---

# get-weather - 

## Description

get-weather parses and displays the current weather and forecast from the yahoo! rss.  simply enter your zipcode (or ixx code from yahoo weather) and -c(elcius) if you don’t want fahrenheit temperatures.

## Comments



## Usage



## TODO



## function

`get-weather`

## Code

`#
 #
 ###################################################################################################
 ###################################################################################################
 #########
 ###################################################################################################
 param($zip=14586,[switch]$celcius,[switch]$Popup)
 $url = "http`://weather.yahooapis.com/forecastrss?p={0}{1}" -f $zip, $(if($celcius){"&u=c"})
 
 $channel = ([xml](New-Object Net.WebClient).DownloadString($url)).rss.channel
 
 function TempColor ($temp) {
    if($celcius) { 
       if( $temp -lt 0 ) { "blue" } elseif( $temp -le 10 ) { "cyan" } elseif( $temp -le 21 ) { "blue" } elseif( $temp -lt 27 ) { "green" } else { "red" } 
    } else { 
       if( $temp -lt 5 ) { "blue" } elseif( $temp -le 50 ) { "cyan" } elseif( $temp -le 70 ) { "blue" } elseif( $temp -lt 80 ) { "green" } else { "red" }
    }
 }
 
 if($channel) {
    if( ($Host.PrivateData.GetType().Name  -eq "PoshOptions") -OR ( $Popup -AND (Get-Command Out-WPF -Type Cmdlet -EA SilentlyContinue))) { 
 
       $template = @"
   <StackPanel xmlns="http`://schemas.microsoft.com/winfx/2006/xaml/presentation" >
   <TextBlock FontFamily="Constantia" FontSize="12pt">{0}, {1} {2}</TextBlock>
   <StackPanel Orientation="Horizontal">
   <StackPanel VerticalAlignment="Top" Margin="6,2,6,2" ToolTip="{3}">
   <Image Source="http`://image.weather.com/web/common/wxicons/52/{4}.png" Stretch="Uniform"
          Width="{{Binding Source.PixelWidth,RelativeSource={{RelativeSource Self}}}}"
          Height="{{Binding Source.PixelHeight,RelativeSource={{RelativeSource Self}}}}" />
   <TextBlock TextAlignment="Center"><Run FontWeight="700" Text="Now: " /><Run Foreground="{5}"> {6}{7}</Run></TextBlock>
   </StackPanel>
   <StackPanel VerticalAlignment="Top" Margin="6,2,6,2" ToolTip="{8}">
   <Image Source="http`://image.weather.com/web/common/wxicons/52/{9}.png" Stretch="Uniform"
          Width="{{Binding Source.PixelWidth,RelativeSource={{RelativeSource Self}}}}"
          Height="{{Binding Source.PixelHeight,RelativeSource={{RelativeSource Self}}}}" />
   <TextBlock TextAlignment="Center">
     <Run FontWeight="700">{10}</Run><LineBreak/>
     <Run Foreground="{11}">{12}{7}</Run> - <Run Foreground="{13}">{14}{7}</Run></TextBlock>
   </StackPanel>
   <StackPanel VerticalAlignment="Top" Margin="6,2,6,2" ToolTip="{15}">
   <Image Source="http`://image.weather.com/web/common/wxicons/52/{16}.png" Stretch="Uniform"
          Width="{{Binding Source.PixelWidth,RelativeSource={{RelativeSource Self}}}}"
          Height="{{Binding Source.PixelHeight,RelativeSource={{RelativeSource Self}}}}" />
   <TextBlock TextAlignment="Center">
   <Run FontWeight="700">{17}</Run><LineBreak/>
   <Run Foreground="{18}">{19}{7}</Run> - <Run Foreground="{20}">{21}{7}</Run></TextBlock>
   </StackPanel>
   </StackPanel>
   </StackPanel>
 "@ 
 
       $template = ($template -f $channel.location.city,  $channel.location.region, $channel.lastBuildDate, 
       $channel.item.condition.text, $channel.item.condition.code, (TempColor $channel.item.condition.temp), $channel.item.condition.temp,
       $channel.units.temperature, $channel.item.forecast[0].text, $channel.item.forecast[0].code, 
       $channel.item.forecast[0].day, (TempColor $channel.item.forecast[0].low), $channel.item.forecast[0].low, (TempColor $channel.item.forecast[0].high), $channel.item.forecast[0].high, 
       $channel.item.forecast[1].text, $channel.item.forecast[1].code, $channel.item.forecast[1].day,
       (TempColor $channel.item.forecast[1].low), $channel.item.forecast[1].low, (TempColor $channel.item.forecast[1].high), $channel.item.forecast[1].high)
 
       $template | out-clipboard
       if($Host.PrivateData.GetType().Name  -eq "PoshOptions") {
          Out-WPF -SourceTemplate $template -Popup:$Popup
       } else {
          Out-WPF -SourceTemplate $template 
       }
 
       
       $current = $channel.item.condition
       Write-Host
       Write-Host ("Location:    {0}" -f $channel.location.city)
       Write-Host ("Last Update: {0}" -f $channel.lastBuildDate)
       Write-Host ("Weather:     {0}" -f $current.text)-NoNewline
       Write-Host (" {0}°{1}" -f $current.temp, $(if($celcius){"C"}else{"F"})) -ForegroundColor $(TempColor $current.temp)
       Write-Host
       Write-Host "Forecasts"
       Write-Host "---------"
       foreach ($f in $channel.item.forecast) {
          Write-Host ("`t{0}, {1}: {2}" -f $f.day, $f.date, $f.text) -NoNewline
          Write-Host (" {0}-{1}°{2}" -f $f.low, $f.high, $(if($celcius){"C"}else{"F"})) -ForegroundColor $(TempColor $f.High)
       }
       Write-Host
    }
 }
 #}
`

