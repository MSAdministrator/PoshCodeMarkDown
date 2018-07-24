---
Author: adrianwoodrup
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6521
Published Date: 2016-09-15t17
Archived Date: 2016-11-18t08
---

# amazon aws user data - 

## Description

this code can be added to an aws instance to set the default password of an ec2 instance. it stops the need for using keys to set the windows password. it needs to be set in the “user data” section when building the instance.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 <powershell>
 
 $ComputerName = $env:COMPUTERNAME
 $user = [adsi]"WinNT://$ComputerName/Administrator,user"
 $user.setpassword("Password")
 
 </powershell>
`

