---
Author: xaegr
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6407
Published Date: 2016-06-27t11
Archived Date: 2016-06-29t10
---

# factorio blog alert - 

## Description

this script plays beep sound when there is new post at factorio blog. converted from js original – https

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 $AlertOnErrors = $false
 $LastGid = '250329347071682810'
 $URI = 'http://api.steampowered.com/ISteamNews/GetNewsForApp/v0002/?appid=427520&count=3&maxlength=300&format=json'
 
 $client = New-Object System.Net.WebClient
 
 function Check()
 {
     try 
     {
         $data = $client.DownloadString($URI)
         $json = ConvertFrom-Json $data
         $currentGid = $json.appnews.newsitems[0].gid
     }
     catch
     {
         Write-Warning "Error: $_"
         if ($AlertOnErrors)
         {
             return $true
         }
     }
     return ($currentGid -ne $LastGid)
 }
 
 while (1)
 {
     if (Check)
     {
         Write-Warning "ALERT!!!"
         [System.Console]::Beep(1000,1000)
     }
     
     sleep -Seconds $Interval
 }
`

