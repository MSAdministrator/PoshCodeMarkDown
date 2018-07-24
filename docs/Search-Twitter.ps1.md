---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2211
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t02
---

# search-twitter.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Search Twitter for recent mentions of a search term
 
 .EXAMPLE
 
 Search-Twitter PowerShell
 Searches Twitter for the term "PowerShell"
 
 #>
 
 param(
     $Pattern = "PowerShell"
 )
 
 Set-StrictMode -Version Latest
 
 Add-Type -Assembly System.Web
 $queryUrl = 'http://integratedsearch.twitter.com/search.html?q={0}'
 $queryUrl = $queryUrl -f ([System.Web.HttpUtility]::UrlEncode($pattern))
 
 $wc = New-Object System.Net.WebClient
 $wc.Encoding = [System.Text.Encoding]::UTF8
 $results = $wc.DownloadString($queryUrl)
 
 $matches = $results |
     Select-String -Pattern '(?s)<div[^>]*msg[^>]*>.*?</div>' -AllMatches
 
 foreach($match in $matches.Matches)
 {
     $tweet = $match.Value -replace '<[^>]*>', ''
 
     [System.Web.HttpUtility]::HtmlDecode($tweet.Trim()) + "`n"
 }
`

