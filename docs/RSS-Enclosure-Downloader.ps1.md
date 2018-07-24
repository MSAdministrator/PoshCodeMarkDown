---
Author: alexander groß
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 632
Published Date: 2008-10-09t20
Archived Date: 2012-10-25t21
---

# rss enclosure downloader - 

## Description

rss enclosure downloader

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $feed=[xml](New-Object System.Net.WebClient).DownloadString("http://the/rss/feed/url")
 
 foreach($i in $feed.rss.channel.item) {
 	$url = New-Object System.Uri($i.enclosure.url)
 
 	$url.ToString()
 	$url.Segments[-1]
 
 	(New-Object System.Net.WebClient).DownloadFile($url, $url.Segments[-1])
 }
`

