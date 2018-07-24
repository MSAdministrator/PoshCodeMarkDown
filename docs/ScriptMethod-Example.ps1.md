---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3719
Published Date: 2013-10-29t18
Archived Date: 2016-05-26t08
---

# scriptmethod example - 

## Description

an example of a script method with mandatory parameters

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $x = New-Object PSObject | Add-Member -MemberType ScriptMethod -Name Test -Value {
     .{
         param (	
             [Parameter(Mandatory=$true)]
             [ValidateNotNullOrEmpty()]
 	        [string]$Message
         )
         "This is the message: $Message"
     } @args 
 } -PassThru
 
`

