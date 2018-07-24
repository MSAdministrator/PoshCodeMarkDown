---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 187
Published Date: 2008-04-26t15
Archived Date: 2017-05-30t19
---

# resolve-url - 

## Description

figure out the real url’s behind the shortened forms created by snipurl, tinyurl, twurl, is.gd, ...

## Comments



## Usage



## TODO



## function

`resolve-url`

## Code

`#
 #
 ###################################
 function Resolve-URL([string[]]$urls) { 
    [regex]$snip  = "(?:https?://)?(?:snurl|snipr|snipurl)\.com/([^?/ ]*)\b"
    [regex]$tiny  = "(?:https?://)?TinyURL.com/([^?/ ]*)\b"
    [regex]$isgd  = "(?:https?://)?is.gd/([^?/ ]*)\b"
    [regex]$twurl = "(?:https?://)?twurl.nl/([^?/ ]*)\b"
 
    switch -regex ($urls) {
       $snip {
          write-output $snip.Replace( $_, (new-object System.Net.WebClient
             ).DownloadString("http://snipurl.com/resolveurl?id=$($snip.match( $_ ).groups[1].value)"))
       }
       $tiny {
          $doc = ConvertFrom-Html "http://tinyurl.com/preview.php?num=$($tiny.match( $_ ).groups[1].value)"
          write-output $tiny.Replace( $_, "$($doc.SelectSingleNode(""//a[@id='redirecturl']"").href)" )
       }
       $isgd {
          $doc = ConvertFrom-Html "http://is.gd/$($isgd.match( $_ ).groups[1].value)-"
          write-output $isgd.Replace( $_, "$($doc.SelectSingleNode(""//div[@id='main']/p/a"").href)") 
       }
       $twurl {
          $doc = ConvertFrom-Html "http://tweetburner.com/links/$($twurl.match( $_ ).groups[1].value)"
          write-output $twurl.Replace($_, "$($doc.selectsingleNode(""//div[@id='main-content']/p/a"").href)" )
       }
       default { write-output $_ }
    }
 }
`

