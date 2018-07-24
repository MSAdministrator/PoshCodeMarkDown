---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3019
Published Date: 2011-10-21t13
Archived Date: 2011-10-23t11
---

# find-command - 

## Description

because people keep asking questions on irc where the answer seems obvious…

## Comments



## Usage



## TODO



## function

`find-command`

## Code

`#
 #
 function Find-Command{
 param([Parameter($Mandatory=$true)]$question)
 
 Get-Command -Verb ($question.Split() | Where {Get-Verb $_ }) -Noun $question.Split()
 }
`

