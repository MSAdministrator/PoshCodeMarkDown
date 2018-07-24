---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6125
Published Date: 2016-12-01t13
Archived Date: 2016-03-18t23
---

# keylogger - 

## Description

example of elementary keylogger.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 [String]$buff = ""
 
 while($true) {
   [Console]::ReadKey("`r") | % {
     if ($_.Key -eq 'Enter') {break}
     if ([Char]::IsLetterOrDigit($_.KeyChar) -or [Char]::IsWhiteSpace($_.KeyChar) -or`
         [Char]::IsPunctuation($_.KeyChar) -or [Char]::IsSymbol($_.KeyChar)) {
       $buff += $_.KeyChar
       Write-Host $_.KeyChar -no
     }
   }
 }
 ""
 
 if (-not [String]::IsNullOrEmpty($buff)) {
   Out-File ($pwd.Path + '\keylogger.log') -in $buff -enc ASCII -app -for
 }
`

