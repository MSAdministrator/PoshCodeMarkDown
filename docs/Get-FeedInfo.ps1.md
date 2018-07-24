---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1698
Published Date: 
Archived Date: 2010-03-23t07
---

# get-feedinfo - 

## Description

takes an array of rss feed urls and gets the site url and title..

## Comments



## Usage



## TODO



## function

`get-feedinfo`

## Code

`#
 #
 function Get-FeedInfo([string[]]$feeds) {
 $blogs=@(); $broken=@(); $feeds = $feeds | sort -unique {$_}
 foreach($feed in $feeds){ 
    try { 
       $xml = Get-WebPageContent $feed;
    } catch { 
       $broken += $feed
       $xml = $null
    }
    if($xml.html.rss) { $xml = $xml.html; write-warning $feed }
    if($xml.rss) {
       $blogs += New-Object PSObject -Property @{ 
          title= $xml.rss.channel.title
          link = $xml.rss.channel.link | ? { $_ -is [string] }
          feed = $feed 
       }
    } elseif($xml.feed) {
       $blogs += New-Object PSObject -Property @{ 
          link = $xml.feed.link |? {$_.rel -eq 'alternate' } | select -expand href
          feed = $feed 
       }
    } else {
       $broken += $feed
       $blogs += New-Object PSObject -Property @{ feed = $feed }
    }
 }
 Write-Output $blogs, $broken
 }
`

