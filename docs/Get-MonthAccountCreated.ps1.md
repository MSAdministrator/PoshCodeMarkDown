---
Author: sepeck
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3976
Published Date: 2013-02-20t17
Archived Date: 2013-03-02t02
---

# get-monthaccountcreated - 

## Description

function get-monthaccountcreated

## Comments



## Usage



## TODO



## function

`get-monthaccountcreated`

## Code

`#
 #
 function Get-MonthAccountCreated {
     <#
     .SYNOPSIS
     Gets Account created in given month.
     .DESCRIPTION
     Gets Account created in given month.
     
     .EXAMPLE 
     Get-MonthAccountCreated -Month 2 -Year 2012
     .EXAMPLE
     Get-MonthAccountCreated -Month 10 -Year 2012
         
     .PARAMETER Month
     Month as number.
     .PARAMETER YEAR
     Year
     #>
     [cmdletBinding()]
     param(
         [Parameter(mandatory=$True)]
         [ValidateRange(1,12)]
         [int]$Month,
         [Parameter(mandatory=$True)]
         [ValidateRange(1990,2020)]
         [int]$Year
     )
  
     BEGIN {}
     PROCESS {
         $mbx = Get-Mailbox -ResultSize Unlimited
         $result = $mbx | Where-Object { ($_.WhenCreated).Month -eq $month -and ($_.WhenCreated).year -eq $year } | Select-Object Name, WhenCreated 
         $result
         }
     
     END{}
 }
`

