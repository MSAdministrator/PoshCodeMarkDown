---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 4523
Published Date: 2013-10-16t09
Archived Date: 2013-10-21t07
---

# check-newgmail - 

## Description

not for regular use, this is just a demo with using com object to check new letters on gmail.

## Comments



## Usage



## TODO



## function

`check-newgmail`

## Code

`#
 #
 function Check-NewGmail {
   param(
     [String]$Email = (Read-Host "Enter your email"),    
     [Security.SecureString]$Password = (Read-Host "Enter email password" -as)
   )
   
   function str([Security.SecureString]$s) {
     return [Runtime.InteropServices.Marshal]::PtrToStringAuto(
       [Runtime.InteropServices.Marshal]::SecureStringToBSTR($s)
     )
   }
   $com = New-Object -com MSXML2.XMLHTTP.3.0
   $com.open('GET', $('https://' + $Email + ':' + `
              (str $Password) + '@mail.google.com/mail/feed/atom'), $false)
   $com.setRequestHeader('Content-Type', 'application/x-www-from-urlcoded')
   $com.send()
   
   $com.responseText -match 'fullcount>\d+' | Out-Null; $res = ($matches[0] -split '>')[1]
   Write-Host You have $res new letter`(s`).
 }
`

