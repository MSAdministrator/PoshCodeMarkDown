---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 260.13
Encoding: ascii
License: cc0
PoshCode ID: 2147
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t19
---

# get-answer.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

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
 
 Uses Bing Answers to answer your question
 
 .EXAMPLE
 
 Get-Answer "(5 + e) * sqrt(x) = Pi"
 Calculation
 (5+e )*sqrt ( x)=pi  : x=0.165676
 
 .EXAMPLE
 
 Get-Answer msft stock
 Microsoft Corp (US:MSFT) NASDAQ
 29.66  -0.35 (-1.17%)
 After Hours: 30.02 +0.36 (1.21%)
 Open: 30.09    Day's Range: 29.59 - 30.20
 Volume: 55.60 M    52 Week Range: 17.27 - 31.50
 P/E Ratio: 16.30    Market Cap: 260.13 B
 
 #>
 
 Set-StrictMode -Version Latest
 
 $question = $args -join " "
 
 function Main
 {
     Add-Type -Assembly System.Web
 
     $encoded = [System.Web.HttpUtility]::UrlEncode($question)
     $url = "http://www.bing.com/search?q=$encoded"
     $text = (new-object System.Net.WebClient).DownloadString($url)
 
     $startIndex = $text.IndexOf('<div class="ans">')
 
     $endIndex = $text.IndexOf('<div class="sn_att2">')
     if($endIndex -lt 0) { $endIndex = $text.IndexOf('<div id="results">') }
 
     if(($startIndex -ge 0) -and ($endIndex -ge 0))
     {
         $partialText = $text.Substring($startIndex, $endIndex - $startIndex)
 
         $partialText = $partialText -replace '<div[^>]*>',"`n"
         $partialText = $partialText -replace '<tr[^>]*>',"`n"
         $partialText = $partialText -replace '<li[^>]*>',"`n"
         $partialText = $partialText -replace '<br[^>]*>',"`n"
         $partialText = $partialText -replace '<span[^>]*>'," "
         $partialText = $partialText -replace '<td[^>]*>',"    "
 
         $partialText = CleanHtml $partialText
 
         $partialText = $partialText -split "`n" |
             Foreach-Object { $_.Trim() } | Where-Object { $_ }
         $partialText = $partialText -join "`n"
 
         [System.Web.HttpUtility]::HtmlDecode($partialText.Trim())
     }
     else
     {
         "`nNo answer found."
     }
 }
 
 function CleanHtml ($htmlInput)
 {
     $tempString = [Regex]::Replace($htmlInput, "(?s)<[^>]*>", "")
     $tempString.Replace("&nbsp&nbsp", "")
 }
 
 . Main
`

