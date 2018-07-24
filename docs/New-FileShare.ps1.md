---
Author: jeffhicks
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 935
Published Date: 2009-03-12t08
Archived Date: 2015-05-07t02
---

# new-fileshare - 

## Description

this topic came up recently in the powershell forum at scriptinganswers.com and i thought you might find it helpful. the issue was creating a file share remotely. in the �olden days�, this would have meant using a command line tool like rmtshare, which is really just fine. but if you have a larger workflow, doing it natively in powershell may be more desirable.

## Comments



## Usage



## TODO



## function

`new-fileshare`

## Code

`#
 #
 Function New-FileShare {
     Param([string]$computername=$env:computername,
           [string]$path=$(Throw "You must enter a complete path relative to the remote computer."),
           [string]$share=$(Throw "You must enter the name of the new share."),
           [string]$comment,
           [int]$connections
           )
           
     $FILE_SHARE = 0
  
     if (! $connections) {
         [void]$connections = $NULL
         }
     
     [wmiclass]$wmishare="\\$computername\root\cimv2:win32_share"
     
     $return=$wmishare.Create($path,$share,$FILE_SHARE,$connections,$comment)
     
     Switch ($return.returnvalue) {
         0 {$rvalue = "Success"}
         2 {$rvalue = "Access Denied"}     
         8 {$rvalue = "Unknown Failure"}     
         9 {$rvalue = "Invalid Name"}     
         10 {$rvalue = "Invalid Level"}     
         21 {$rvalue = "Invalid Parameter"}     
         22 {$rvalue = "Duplicate Share"}     
         23 {$rvalue = "Redirected Path"}     
         24 {$rvalue = "Unknown Device or Directory"}
         25 {$rvalue = "Net Name Not Found"}
     }
     
     if ($return.returnvalue -ne 0) {
         Write-Warning ("Failed to create share {0} for {1} on {2}. Error: {3}" -f $share,$path,$computername,$rvalue) 
         return $False
     }
     else {
         return $True
     }
 }
`

