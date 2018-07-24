---
Author: jack neff
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4685
Published Date: 2013-12-09t20
Archived Date: 2013-12-13t12
---

# autorefreshwebpage - 

## Description

automatically refreshes a webpage.  only works in internet explorer…sorry.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $url = "http://www.somepage.com/"
 $interval = 60
 $shell = New-Object -ComObject Shell.Application
 
 "Refreshing $url every $interval seconds."
 "Press ctrl+c to stop."
 
 while(1){
     if (($shell.Windows() | where LocationURL -eq $url) -eq $null) { start $url }
     ($shell.Windows() | where LocationURL -eq $url).Refresh()
     sleep $interval
 }
`

