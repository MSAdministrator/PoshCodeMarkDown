---
Author: nathan kasco
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 6189
Published Date: 2016-01-26t00
Archived Date: 2016-05-17t10
---

# type-typewriter - 

## Description

make write-host text appear as if it is being typed on a typewriter

## Comments



## Usage



## TODO



## function

`type-typewriter`

## Code

`#
 #
 function Type-Typewriter
 {
 <#
 .Synopsis
    Make write-host text appear as if it is being typed on a typewriter
 .DESCRIPTION
    Input text and if desired specify the write speed (25-250 milliseconds) for the text
 .EXAMPLE
    Type-Typewriter "Hello world!"
 .EXAMPLE
    Type-Typewriter "Hello world!" 250
 .NOTES
    v1.0 - 2016-01-25 - Nathan Kasco
 #>
 
     [CmdletBinding()]
     [Alias()]
     [OutputType([string])]
 
     Param
     (
         [Parameter(Mandatory=$true, Position=0)]
         [alias("Name")]
         [string]
         $text,
         
         [Parameter(Mandatory=$false, Position=1)]
         [ValidateRange(25,250)]
         [int]
         $typeSpeed = 125
     )
 
     function sleep-host{
         Start-Sleep -Milliseconds $typeSpeed
     }
 
     $count = 0
     while($count -lt $text.Length){
         Write-Host $text.Chars($count) -NoNewline
         sleep-host
         $count++
     }
 }
`

