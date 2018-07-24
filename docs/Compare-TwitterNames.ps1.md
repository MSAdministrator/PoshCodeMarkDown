---
Author: steven murawski
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 871
Published Date: 2009-02-16t07
Archived Date: 2017-03-12t01
---

# compare-twitternames.ps1 - 

## Description

this script will compare the names of the people you follow on twitter and the people following you.  it returns a comparison object consisting of the twitter name of a subject and a side indicator – “<=” means that you are following a subject who is not following you, “=>” means that you are followed by someone who you are not following.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #"<=" means that you are following a subject who is not following you, 
 #"=>" means that you are followed by someone who you are not following.
 
 function GetTwitterNames([string]$query)
 {   
     $wc = new-object System.Net.WebClient
     $wc.Credentials = $script:credential.GetNetworkCredential()
 
     $nbrofpeople = 0
     $page = "&page="
     $names = @()
 
     do 
     {
         $url = $query
         if ($nbrofpeople -gt 0)
         {
             $url = $url+$page+($nbrofpeople/100 +1)
         }
 
         [xml]$nameslist = $wc.DownloadString($url)
 
         $names += $nameslist.users.user | select name
 
         $nbrofpeople += 100
     } while ($names.count -eq $nbrofpeople)
 
     return $names
 }
 
 $twitter = "http://twitter.com/statuses/"
 $friends = $twitter + "friends.xml?lite=true"
 $followers = $twitter + "followers.xml?lite=true"
 
 $credential = Get-Credential
 
 $friendslist = GetTwitterNames($friends)
 $followerslist = GetTwitterNames($followers)
 
 $sync = 0
 if ($friendslist.count -gt $followerslist.count)
 {
 	$sync = ($friendslist.count)/2
 }
 else
 {
 	$sync = ($followerslist.count)/2
 }
 
 $Status = @{Name='Status';Expression={if ($_.sideindicator -like '=>') {'Followed By'} else {'Following'}}}
 
 compare-object $friendslist $followerslist -SyncWindow ($sync) -Property name | Select-Object Name, $Status
`

