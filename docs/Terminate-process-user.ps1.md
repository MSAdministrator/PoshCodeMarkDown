---
Author: ty lopes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3689
Published Date: 2012-10-12t08
Archived Date: 2012-10-14t23
---

# terminate process / user - 

## Description

#ty lopes – calgary – oct 2012

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #(Troy is a huge nerd)
 
 
 $username = "username"
 $process= "notepad"
 
 $owners = @{}
 gwmi win32_process |% {$owners[$_.handle] = $_.getowner().user}
 get-process $process | select processname,Id,@{l="Owner";e={$owners[$_.id.tostring()]}} | where-object {$_.owner -eq $username} | kill -force
`

