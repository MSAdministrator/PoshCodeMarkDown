---
Author: jan egil ring
Publisher: 
Copyright: 
Email: jer@powershell.no
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1605
Published Date: 
Archived Date: 2010-01-27t03
---

# get-lotterynumbers - get-lotterynumbers.ps1

## Description

generates lottery numbers based on user input.

## Comments

generates lottery numbers based on user input.

## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################
 #
 #
 #
 #
 #
 #
 ###########################################################################
 
 $maximumnumber = Read-Host "How many numbers are available in the lottery?"
 $numbercount = Read-Host "How many numbers are drawn?"
 $numbers = @()
 do {
 $number = Get-Random -Minimum 1 -Maximum $maximumnumber
 if ($numbers -notcontains $number)
 {
 $numbers += $number
 }
 }
 until ($numbers.count -eq $numbercount)
 $numbers | Sort-Object
`

