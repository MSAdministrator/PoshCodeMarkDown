---
Author: forthebetterr
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4649
Published Date: 2013-11-26t13
Archived Date: 2013-12-06t18
---

# forthebetterr - 

## Description

forthebetterr

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 param ([String] $ScreenName)
 
 $client = New-Object System.Net.WebClient
 $idUrl = "https://api.twitter.com/1/users/show.json?screen_name=me"
 $data = $client.DownloadString($idUrl)$ScreenNa
 
 $start = 0
 
 $findStr = '"id":'
 do {
     $start = $data.IndexOf($findStr, $start + 1)
     if ($start -gt 0) {
         $start += $findStr.Length
         $end = $data.IndexOf(',', $start)
         $userId = $data.SubString($start, $end-$start)
     }
 } while ($start -le $data.Length -and $start -gt 0)
 
 $feed = "http://twitter.com/statuses/user_timeline/$userId.rss"
 
 $feed
`

