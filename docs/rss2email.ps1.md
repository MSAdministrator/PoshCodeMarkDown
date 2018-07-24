---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 742
Published Date: 
Archived Date: 2008-12-21t05
---

# rss2email - 

## Description

generate an email from an rss feed and store the feed item in a cache file to support emailing only new feed items since last execution.

## Comments



## Usage



## TODO



## script

`get-md5`

## Code

`#
 #
 param ($smtpServer, $rssCacheFile, $recipients, $rssUrl)
 
 #######################
 function Get-MD5
 {
     param ($str)
     $cryptoServiceProvider = [System.Security.Cryptography.MD5CryptoServiceProvider]
 
     $hashAlgorithm = new-object $cryptoServiceProvider
     $byteArray = [Text.Encoding]::UTF8.GetBytes($str)
     $hashByteArray = $hashAlgorithm.ComputeHash($byteArray)
     $hex = $null
     $hashByteArray | foreach {$hex += $_.ToString("X2")}
     $hex
 
 
 #######################
 function Get-Rss
 {
     $feed = [xml](new-object System.Net.WebClient).DownloadString($rssUrl)
     $feed.rss.channel.Item | foreach {$_.description = ""}
     $results = $feed.rss.channel.Item | Out-String
     $title = $feed.rss.channel.title
     @{$(Get-MD5 $results) = @($title,$results)}
 
 
 #######################
 function Send-RssEmail
 {
     param ($recipients,$subject,$message)
     $smtpmail = [System.Net.Mail.SMTPClient]("$smtpServer")
     $smtpmail.Send($recipients, $recipients, $subject, $message)
 
 
 #######################
 function Get-RssCache
 {
     if (test-path $rssCacheFile) {
        $guidcache = Get-Content $rssCacheFile
     } else {
        $guidcache = @()
     }
 
     return $guidcache
 
 
 #######################
 $rss = $(Get-Rss $rssUrl)
 
 $guidcache = $(Get-RssCache)
 
 if (!($guidcache -contains $rss.keys))
 {
    Send-RssEmail $recipients $($rss.values)[0] $($rss.values)[1]
    $rss.keys >> $rssCacheFile
 }
`

