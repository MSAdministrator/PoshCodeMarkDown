---
Author: archdeacon
Publisher: 
Copyright: 
Email: 
Version: 2.1
Encoding: ascii
License: cc0
PoshCode ID: 3196
Published Date: 2013-01-27t22
Archived Date: 2013-01-02t04
---

# debug regex match - 

## Description

debug regex operation when matching any string

## Comments



## Usage



## TODO



## function

`debug-regex`

## Code

`#
 #
 function Debug-Regex {
 <#
 .SYNOPSIS
 A very simple function to debug a Regex search operation and show any 'Match' 
 results. (V2.1 Export Alias db & some patterns, 28 Jan 2012).
 
 .DESCRIPTION
 Sometimes it is easier to correct any regex usage if each match can be shown 
 in context. This function will show each successful result in a separate 
 colour, including the strings both before and after the match. Using the -F
 switch will allow only the first of any matches to be returned.
 Commonly used regex patterns may be added to this module, 'exported' and then 
 assigned to a variable before being passed to the function. (They can also be
 used with the Select-String Cmdlet).
 
 .EXAMPLE
 Debug-Regex '\b[A-Z]\w+' 'Find capitalised Words in This string' -first
 Use the -F switch to return only the first match. Produces the following...
 
 MATCHES  ---------------<match>--------------------
 1 [<Find> capitalised Words in This string]
 .EXAMPLE
 Debug-Regex '\b[0-9]+\b' 'We have to find numbers like 123 and 456 in this'
 Produces the following...
 
 MATCHES  ---------------<match>--------------------
 1 [We have to find numbers like <123> and 456 in this]
 2 [We have to find numbers like 123 and <456> in this]
 
 .EXAMPLE
 PS >$a = Get-Variable regexDouble -ValueOnly
 PS >db $a 'Find some double words words in this this string'
 Assign the desired Regex string to $a and use this and the alias as parameters.
  
 .NOTES
 Based on a suggestion from the book 'Mastering Regular Expressions' by J.Friedl,
 page 429.
 
 .LINK
    Web Address Http://www.SeaStarDevelopment.BraveHost.com 
 #>
    param ([Parameter(mandatory=$true)][regex]$regex,
           [Parameter(mandatory=$true)][string]$string,
           [switch]$first)
    $m = $regex.match($string)
    if (!$m.Success) {
       Write-Host "No Match using Regex '$regex'" -Fore Yellow
       return
    }
    $count = 1
    Write-Host "MATCHES  --------------------<" -Fore Yellow -NoNewLine
    Write-Host "match"                          -Fore White  -NoNewLine
    Write-Host ">--------------------"          -Fore Yellow
    while ($m.Success) {
       Write-Host "$count $($m.result('[$`<'))" -Fore Yellow -NoNewLine 
       Write-Host "$($m.result('$&'))"          -Fore White  -NoNewLine 
       Write-Host "$($m.result('>$'']'))"       -Fore Yellow 
       if ($first) {
          return
       }
       $count++
       $m = $m.NextMatch()
    }
 }
 
 Set-Variable -Name regexDouble  -value '\b(\w+)((?:\s|<[^>]+>)+)(\1\b)' `
              -Description 'Find double word pairs within <>'
 Set-Variable -Name regexNumbers -value '\b[0-9]+\b' `
              -Description 'Find only numbers in a string'
 
 New-Alias db Debug-Regex
 Export-ModuleMember -Function Debug-Regex -Variable regex* -Alias db
`

