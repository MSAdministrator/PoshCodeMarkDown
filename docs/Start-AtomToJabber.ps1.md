---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 179
Published Date: 
Archived Date: 2009-11-28t08
---

# start-atomtojabber - 

## Description

a simpler version of http

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ##########################################################################################
 ##########################################################################################
 param (
     $JabberId = $( Read-Host "Bot's Jabber ID" )
    ,$Password = $( Read-Host "Bot's Password" -asSecure)
    ,$AtomFeeds[] = @("http://groups.google.com/group/microsoft.public.windows.powershell/feed/atom_v1_0_topics.xml")
 )
 $ErrorActionPreference = "Stop"
 
 $global:PoshXmppClient = 
 PoshXmpp\Connect-Chat $Chat $ChatNick
 
 $feedReader = new-object Xml.XmlDocument
 
 "PRESS ANY KEY TO STOP"
 while(!$Host.UI.RawUI.KeyAvailable) {
    "Checking feeds..."
    foreach($feed in $AtomFeeds) {
       $feedReader.Load($feed)
       for($i = $feedReader.feed.entry.count - 1; $i -ge 0; $i--) {
          $e = $feedReader.feed.entry[$i]
          if([datetime]$e.updated -gt $global:LastNewsCheck) {
             [Threading.Thread]::Sleep( 1000 )
          }
       }
       if( [datetime]$feedReader.feed.entry[0].updated -gt $nnc ) {
          $nnc = [datetime]$feedReader.feed.entry[0].updated
       }
    }
    $counter = 0
    while(!$Host.UI.RawUI.KeyAvailable -and ($counter++ -lt 600)) {
       [Threading.Thread]::Sleep( 1000 )
    }
 }
 
 $global:PoshXmppClient.Close()
`

