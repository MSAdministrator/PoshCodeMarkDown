---
Author: qodosh
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 4903
Published Date: 2014-02-16t05
Archived Date: 2017-05-22t01
---

# getmacs.ps1 - 

## Description

very basic script that gets mac addresses (netbios table) from a list of remote hosts.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###############################################################################################################################
 #
 $Servers = Get-Content 'C:\servers.txt'
 $log = 'c:\log.txt'
 foreach ($Server in $Servers)
 {
     nbtstat -a $Server >> $log
 } 
 start $log
`

