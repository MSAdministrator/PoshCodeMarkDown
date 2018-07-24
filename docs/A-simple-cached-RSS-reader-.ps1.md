---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 709
Published Date: 
Archived Date: 2009-01-06t12
---

#  - 

## Description

a simple cached rss reader. fetches rss feeds, displays mutliple feeds merged in date order, opens items in browser.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ####################################################################################################
 ####################################################################################################
 PARAM( 
    [string[]]$RssReaderUrls = $(do{
                            $k = Read-Host "Specify an RSS Feed URL (hit Enter when done)"
                            if($k) {write-output $k}
                         } while($k) )
 )
 
 function global:Get-RSS {
    $webClient = New-Object System.Net.WebClient
    $global:RssReaderNews = $RssReaderUrls | % { [xml]$webClient.DownloadString( $_ ) } | 
                             % { $_.rss.channel.item } | 
                             Sort { [DateTime]$_.pubDate } -Desc
    Show-RSS
 }
 
 function global:Show-RSS($days=7) {
    $script:index = -1
    $global:RssReaderNews | Where { ([DateTime]$_.pubDate).AddDays($days) -gt [DateTime]::Now } | 
            ft @{l="Index";e={return $script:index++}},@{l="Day";e={$_.pubDate.split(",",2)[0]}}, title, link -auto
 }
 
 function global:Open-RssItem($index=0) {
    [Diagnostics.Process]::Start( $($global:RssReaderNews[$index].link) )
 }
 $global:RssReaderUrls = $RssReaderUrls
 Get-RSS
`

