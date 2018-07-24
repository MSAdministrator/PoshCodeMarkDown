---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1300
Published Date: 
Archived Date: 2010-01-04t22
---

# get-twitterreply - 

## Description

get-twitterreply modification get all replies

## Comments



## Usage



## TODO



## function

`get-twitterreply`

## Code

`#
 #
 
 Function Get-TwitterReply { 
  param ($username, $password)
  if ($WebClient -eq $null) {$Global:WebClient=new-object System.Net.WebClient  }
  $WebClient.Credentials = (New-Object System.Net.NetworkCredential -argumentList $username, $password)
  $page = 0
  $ret = @()
  do {  	$Page ++
     $replies = ([xml]$webClient.DownloadString("https://twitter.com/statuses/replies.xml?page=$Page")  ).statuses.status
     $ret += $replies
     Write-host  $replies.count
  $ret
  write-host $ret.count
 }
`

