---
Author: boggers
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4103
Published Date: 2015-04-12t17
Archived Date: 2015-10-27t12
---

# sync-time - 

## Description

syncs the system time with that of a remote time server.  uses netcmdlets.

## Comments



## Usage



## TODO



## function

`sync-time`

## Code

`#
 #
 function sync-time(
 [string] $server = "sync-time 0.pool.ntp.org, clock.psu.edu",
 [int] $port = 37)
 {
   $servertime = get-time -server $server -port $port -set
   write-host "Server time:" $servertime 
   write-host "Local time :" $(date)
 }
`

