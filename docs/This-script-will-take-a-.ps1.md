---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4923
Published Date: 
Archived Date: 2014-05-06t23
---

#  - 

## Description

this script will take a twitter user’s screen name and get their rss feed of posts

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
 $idUrl = "https://api.twitter.com/1/users/show.json?screen_name=$ScreenName"
 $data = $client.DownloadString($idUrl)
 
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

