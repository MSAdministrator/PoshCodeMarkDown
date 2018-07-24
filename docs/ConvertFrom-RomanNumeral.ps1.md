---
Author: vince ypma
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 5659
Published Date: 2016-01-03t01
Archived Date: 2016-05-19t06
---

# convertfrom-romannumeral - 

## Description

i have encountered “real world” uses for functions like this.  as an exercise i

## Comments



## Usage



## TODO



## function

`convertfrom-romannumeral`

## Code

`#
 #
 function ConvertFrom-RomanNumeral
 {
   <#
     .SYNOPSIS
         Converts a roman numeral to a number.
     .DESCRIPTION
         Converts a roman numeral - in the range of I..MMMCMXCIX - to a number.
     .PARAMETER Numeral
         A roman numeral in the range I..MMMCMXCIX (1..3,999).
     .INPUTS
         System.String
     .OUTPUTS
         System.Int32
     .NOTES
         Requires PowerShell version 3.0
     .EXAMPLE
         ConvertFrom-RomanNumeral -Numeral MMXIV
     .EXAMPLE
         "MMXIV" | ConvertFrom-RomanNumeral
   #>
     [CmdletBinding()]
     [OutputType([int])]
     Param
     (
         [Parameter(Mandatory=$true,
                    HelpMessage="Enter a roman numeral in the range I..MMMCMXCIX",
                    ValueFromPipeline=$true,
                    Position=0)]
         [ValidatePattern("(?x)^
                 $")]
         [string]
         $Numeral
     )
 
     Begin
     {
         $RomanToDecimal = [ordered]@{
             M  = 1000
             CM =  900
             D  =  500
             CD =  400
             C  =  100
             XC =   90
             L  =   50
             X  =   10
             IX =    9
             V  =    5
             IV =    4
             I  =    1
         }
     }
     Process
     {
         $roman = $Numeral + '$'
         $value = 0
 
         do
         {
             foreach ($key in $RomanToDecimal.Keys)
             {
                 if ($key.Length -eq 1)
                 {
                     if ($key -match $roman.Substring(0,1))
                     {
                         $value += $RomanToDecimal.$key
                         $roman  = $roman.Substring(1)
                         break
                     }
                 }
                 else
                 {
                     if ($key -match $roman.Substring(0,2))
                     {
                         $value += $RomanToDecimal.$key
                         $roman  = $roman.Substring(2)
                         break
                     }
                 }
             }
         }
         until ($roman -eq '$')
 
         $value
     }
     End
     {
     }
 }
`

