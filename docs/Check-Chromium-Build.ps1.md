---
Author: david "makovec" moravec
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 840
Published Date: 
Archived Date: 2009-02-05t17
---

# check chromium build - 

## Description

check latest chromium build

## Comments



## Usage



## TODO



## script

`check-latestchromium`

## Code

`#
 #
 #
 #
 #
 #
 #################################################################
 
 function Check-LatestChromium {
 
 	$url = 'http://build.chromium.org/buildbot/snapshots/chromium-rel-xp/'
 	$XPathRelDate = "//tr[position()=last()-2]//td[3]"
 	$XPathBuild = "//tr[position()=last()-2]//td[2]//a"
 	
 	$page = Invoke-Http get $url
 	
 	$releaseDate = $page | Receive-Http text $XPathRelDate
 	($page | Receive-Http text $XPathBuild) -match "(?<build>\d*)" | Out-Null	
 	
 	"Latest Build is: {0}, released at {1}" -f $matches.build, $releaseDate 
 
 }
`

